---
layout: post
title: 采用 atom 来编写 Jekyll 博客是一个近乎完美的解决方案
date: 2016-09-09T11:01:43.000Z
categories:
  - atom jekyll
---

编写技术博客要考虑的几个点：

- 要云和本地，多机备份方便：Github
- 要格式简单不过度依赖某个固定的格式：Markdown

前面两点确定了 Github + Jekyll 的解决方案。然后接下来是用哪个编辑器呢：

- 能够方便的 push git
- 可以本地很方便的预览编辑 Markdown

考虑过以下很多的方案：

- Markdownpad，支持可视化编辑 Markdown，对 git 不友好
- Sublime Text，对 Markdown 和 git 好像都不太友好
- Eclipse 和 Inteliji，对 git 友好，但是对 Markdown 不友好

最后，atom 可以完美的通过各自插件解决这个问题：

1. 原生支持 Markdown，可以通过 Ctrl+Shift+M 调出来 Markdown 预览页面
2. 可以通过 git-plus 来同 github 通信
3. 可以通过 sync-on-save 来配置成自动在保存文件的时候同步到 github
4. 可以通过 markdown-writer 来优化博客的编写，这个插件可以定制 jekyll 的编写引擎，方便尤其是在图片、表格和超链接方面更加的方便了
5. markdown-writer 还是不够图形化，如果再安装一个 tool-bar-markdown-writer 就屌了，不过这个插件需要配合 tool-bar 这个插件来使用
6. jekyll 可以更加方便的使用 jekyll

上述方案过程中遇到的坑如下：

- git-plus 和 sync-on-save 都是通过调用原生的 git 命令来操作的，所以使用前首先需要配置好 git.exe 的路径给他们。可以通过配置 windows 的 PATH 环境变量来完成
- Github windows 的客户端默认是通过 https 来拉取仓库的，而 git-plus 和 sync-on-save 在通过 https push 的时候会有问题（主要是在输入用户名密码的时候不兼容 windows），所以要通过 ssh 免密码来同步：
    - 自己用 Git-bash 通过 ssh 的方式来将仓库拉下来，这样后续 push 的时候也是通过 ssh 了
    - ssh 免密码：通过 Git-bash 查看 ~/.ssh 下面有没有生成好的私钥，如果有就算了，没有就自己生成，同步到 Github 网站。用 ssh -T git@github.com 来测试连接
    - sync-on-save 插件有一个 bug，push 的时候会提示 git: not a command for git, 解决方法：通过 Ctrl+Alt+I 调出来调试窗口，然后将文件：command-runner.coffee 里面的

      ``` coffee
      getArgs: ->
        if @_needsWorkaround()
          [
            "-c",
            [@command].concat(@args.map((i) ->
              "\"" + i.replace(/\"/g, "\\\"") + "\""
            )).join(" ")
          ]
        else
          [@command]
      ```
    改成
      ``` coffee
        getArgs: ->
        #if @_needsWorkaround()
        #  [
        #    "-c",
        #    [@command].concat(@args.map((i) ->
        #      "\"" + i.replace(/\"/g, "\\\"") + "\""
        #    )).join(" ")
        #  ]
        #else
        #  [@command]
         @args
      ```
    就可以了
- 内外 ssh 到外网是有防火墙的，需要配置代理，具体参看内外文章
- Jekyll 插件有一个 nodejs 的问题，就是 build.coffee 文件里面是用 childProcess.spawn 来执行 jekyll build 命令的，而这个方法在 windows 下面按它的方法有一些问题（可以 google 一下 childProcess.spawn 和 childProcess.exec 的区别），所以在配置 Build Command 的时候要配置成：
  ```
  cmd.exe, /c, C:\RailsInstaller\Ruby2.1.0\bin\jekyll, build
  ```
除此之外，build.coffee 文件里面，要改成：
  ``` coffee
  buildCommand = process.jekyllAtom.buildCommand
  #buildCommand = (process.jekyllAtom.config?.atom?.buildCommand || process.jekyllAtom.buildCommand)
  ```



[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com
