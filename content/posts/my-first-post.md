+++
date = '2025-05-15T16:36:51+08:00'
draft = false
title = '时隔多年，我又搭建了一个 blog post'
summary = '记录一下搭建过程吧'
+++

创建一个新的 post

``` bash
hugo new content content/posts/my-first-post.md
```

在 push 之前记得 `hugo serve`

记录一下搭建过程，方便以后的自己能够简单复现

这次的技术选型是 hugo + PaperMode theme. 

### Install Hugo

直接用 winget 就能很快装好

``` bash
# install
winget install Hugo.Hugo.Extended

# uninstall
winget uninstall --name "Hugo (Extended)"
```

### PaperMode theme

```
# download theme as git submodule
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod

# add to toml, if already have, need to change
echo "theme = 'PaperMod'" >> hugo.toml
```

如果从零开始的话，需要按照 hugo 的教程 [link](https://gohugo.io/host-and-deploy/host-on-github-pages) 配置一下 github action

注意 action 的触发是基于 main branch 的 push。有时候 main branch 的名字会变成 master branch。需要稍微改一下官方文档里边的 yaml 才能正常触发。



