+++
date = '2026-11-05T12:00:00-06:00'
draft = false
title = 'REG: Rectified Gradient Guidance for Conditional Diffusion Models'
summary = 'Guidance 并不从它声称的目标分布采样。合法的目标是调整整条去噪链，现有 guidance 是看不到未来时的近似，REG 再加一个 Jacobian 项去接近精确修正。'
tags = ['Diffusion', 'Guidance']
math = true

[paper]
  venue = 'ICML 2025'
  authors = 'Zhengqi Gao, Kaiwen Zha, Tianyuan Zhang, Zihui Xue, Duane S. Boning'
  arxiv = 'https://arxiv.org/abs/2501.18865'
  code = 'https://github.com/zhengqigao/REG'
+++

## 一句话总结

Guidance 并不是从通常所说的、按 reward 调整后的分布中采样。这篇论文把理论建立在整条去噪链的联合分布上，说明现有 guidance 是看不到未来时对精确修正的近似，再加上一个 Jacobian 项 REG，让结果更接近精确修正。

## 问题

通常对 guidance 的解释是从一个调整后的分布中采样，reward $R_0$ 提高与条件 $y$ 相符的样本的权重。这里 $t=0$ 是数据端，$t=T$ 是纯噪声。

$$
\bar p_\theta(x_0 \mid y) \propto p_\theta(x_0 \mid y)\, R_0(x_0, y)
$$

Classifier guidance 的 reward 是分类器概率的 $w$ 次方，CFG 是条件密度与无条件密度之比的 $w$ 次方，autoguidance 是与一个更弱模型的密度之比的 $w$ 次方。

论文指出这个解释和实际做法对不上。从这个目标推出的修正网络只作用在 $t=0$，而实际做法恰好在 $t=0$ 以外的每一步都修改网络。要求每一步的边缘分布都被调整也不行。$t$ 步的 reward 一旦确定，去噪过程就已经决定了 $t-1$ 步的 reward，所以约束太多，无法同时满足。如果起点噪声仍是标准高斯，所有 reward 都只能是常数。

## 方法

解决办法是用最终样本的 reward 去调整整条链的联合分布：

$$
\bar p_\theta(x_{0:T} \mid y) \propto p_\theta(x_{0:T} \mid y)\, R_0(x_0, y)
$$

它在 $t=0$ 的边缘分布和原来的目标相同，所以样本的目标分布没有变，但现在存在一个合法的采样过程。关键的量是给定当前状态时最终 reward 的期望：

$$
E_t(x_t, y) = \mathbb{E}\left[ R_0(x_0, y) \mid x_t, y \right]
$$

定理 4.1 证明新的转移核是唯一的，每一步的边缘分布都按 $E_t$ 调整，精确的修正噪声预测是：

$$
\bar\epsilon^{\star}_{\theta,t} = \epsilon_{\theta,t} - \sqrt{1-\bar\alpha_t}\, \nabla_{x_t} \log E_t(x_t, y)
$$

定理还说明起点噪声也应该按 $E_T$ 调整。当 $E_T$ 随 $x_T$ 变化很慢时，起点仍近似是标准高斯。

计算 $E_t$ 需要把整条链跑到终点，采样时拿不到。把 $\nabla \log E_t$ 换成局部的 $\nabla \log R_t$，正好就是实际使用的 guidance：

$$
\bar\epsilon_{\theta,t} = \epsilon_{\theta,t} - \sqrt{1-\bar\alpha_t}\, \nabla_{x_t} \log R_t(x_t, y)
$$

这是论文最主要的解释：现有 guidance 是看不到未来时对精确修正的近似。定理 4.2 在确定性采样器下给出误差上界，越接近数据端上界越小。定理 4.3 说明即使取期望，这个近似仍有偏差。

REG 保留了精确解里多一项信息。用确定性采样器时，$E_t$ 就等于 $R_0$ 在预测的干净样本 $\hat x_0$ 处的值，链式法则会带出 Jacobian：

$$
\nabla_{x_t} \log R_0(\hat x_0, y) = \left( \frac{\partial \hat x_0}{\partial x_t} \right)^{\top} \nabla_{\hat x_0} \log R_0(\hat x_0, y), \qquad \hat x_0 = \frac{x_t - \sqrt{1-\bar\alpha_t}\, \epsilon_{\theta,t}}{\sqrt{\bar\alpha_t}}
$$

论文把这个 Jacobian 当作对角矩阵，用噪声预测各分量之和的梯度来估计它，再用 $\nabla \log R_t$ 代替 $\nabla \log R_0$：

$$
\bar\epsilon^{\mathrm{REG}}_{\theta,t} = \epsilon_{\theta,t} - \sqrt{1-\bar\alpha_t}\, \nabla_{x_t} \log R_t(x_t, y) \odot \left( 1 - \sqrt{1-\bar\alpha_t}\, \frac{\partial (\mathbf{1}^{\top} \epsilon_{\theta,t})}{\partial x_t} \right)
$$

每个做 guidance 的步骤要多一次对网络的反向传播。

## 实验结果

- **一维和二维合成数据**：这里可以算出精确的 $\nabla \log E_t$。在大多数网格点上，REG 比普通 CFG 更接近它，各时间步的胜率在 57% 到 74% 之间。
- **ImageNet，64 到 512 像素，DiT-XL/2 和 EDM2**：REG 让 vanilla、cosine、linear、interval CFG 和 autoguidance 的 FID 都下降或持平。DiT-XL/2 加 vanilla CFG 从 2.21 降到 2.04，512 像素的 EDM2-XXL 从 1.83 降到 1.74。EDM2 上的 interval CFG 只在约 20% 的步骤做 guidance，FID 没有变化。
- **COCO 5K 文生图，SD-v1-4 和 SD-XL**：所有设置下 FID 和 CLIP score 都有改善。SD-v1-4 加 vanilla CFG 的 FID 从 20.27 降到 19.63。
- **开销**：运行时间增加到 1.2 到 1.9 倍。内存增加到 1.2 到 1.7 倍，只有 1024 像素的 SD-XL 达到 2.8 倍。

## 我的看法

- 理论比方法更重要。此前在高斯混合例子里已经发现 CFG 并不从它声称的目标分布采样，这篇论文解释了原因：实际做法是看不到未来时对精确修正的近似。
- 把 Jacobian 当作对角矩阵是一个经验做法，只有实验支持。把 reward 梯度固定住时，完整的 Jacobian 乘向量也只需要一次反向传播。我想确认论文为什么说它代价过高，以及用完整的乘积效果会不会更好。
- 这套理论假设去噪过程是马尔可夫链。把它推广到流模型的 ODE，以及像 Momentum Guidance 这样带状态的采样器，就是我的想法[修正动量引导](https://jamessand.github.io/ideas/rectified-momentum-guidance/)要回答的问题。
- 起点分布应该怎样调整还没有解决，目前也不清楚忽略它什么时候会损害样本质量。
