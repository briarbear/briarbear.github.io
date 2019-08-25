---
title: Hexo+GitPage+Gitments+yilia博客搭建手记
date: 2018-06-22 14:58:08
tags: blog
toc: true
categories: 随笔

---

---

> 说明: 经过两天的折腾, 也算把这个事情搞定了,顺手写下这边手记.笔者之前已经使用`SpringBoot`搭建了个人博客站点, [博客预览](http://www.briarbear.cn) . 完整代码已开源,可见博客站点的说明页. 而之所以又着手搭建这个站点,主要考虑到,那个服务器租期为一年,后期维护麻烦,为减少成本,打算慢慢迁移至此. 这边随笔主要记录使用Hexo+GitHub搭建博客的流程,希望对您有所帮助,喜欢点个Star,谢谢.

## 1.  主要步骤  

- 安装`Git`、 `Node.js`(本文示例为Windows平台)
- 创建`Github`仓库,开启`GitPage`
- 安装、使用`Hexo`,更换主题
- 图床、 `Markdown编辑器`
- 多端同步，`Gitment`评论等其他问题

<!-- more -->

---

## 2. 安装Git、Node.js

- 安装`Git`，本文就不详述步骤，[具体可参考-廖雪峰Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
- 安装`Node.js`，[下载网站](https://nodejs.org/en/download/)，笔者下载的是`LTS-Windows64位 5.6.0`安装版本
- 安装时注意保持勾选 `add to path`，任意目录右键 `Git Bash Here`，输入 `npm -v`，检验安装是否成功，显示版本号，则表示正常

----

## 3. 创建GitHub仓库，设为GitPages

- 创建一个仓库，命名为 `用户名.github.io`,如 `briarbear.github.io`

- 这样的命名即为开启 `GitPages`，点开仓库的`setting`，可以看到`GitHub Pages`说明

  ![](http://cdn.briarbear.cn/201806221530_904.png)

- 则博客站点域名即为 `用户名.github.io`

---

## 4. 安装Hexo

- 安装教程可参看 [官方文档](https://hexo.io/zh-cn/docs/)，中文版，详尽易懂 
- 主要步骤为 
  - 安装： `npm install -g hexo-cli`  
  - 初始化：
```shell
$ hexo init <folder>
$ cd <folder>
$ npm install
```

  - 配置：重点配置`deploy` ，使用`Git`同步方式与`GItHub`同步，具体详见 [官方说明](https://hexo.io/zh-cn/docs/configuration.html)

    如笔者的配置，完整的详细配置，可见文末本博客站点的开源：
    
```yaml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy: 
type: git
repo: git@github.com:briarbear/briarbear.github.io.git
branch: master
```

  - 安装`Git`插件：`npm install hexo-deployer-git --save`
- 启动：

  - 此时就可以依次输入命令； `hexo clean`， `hexo generate`, `hexo server`
  - 即本地启动完成：在浏览器输入：`http://localhost:4000`,即可以看到初始效果

  ---

## 5. 使用Hexo

- 常用命令:

```yaml
hexo init [folder] # 新建一个网站。如果没有设置 folder ，Hexo 默认在目前的文件夹建立网站。

hexo new [layout] <title>
#新建一篇文章。如果没有设置 layout 的话，默认使用 _config.yml 中的 default_layout 参数代替。如果标题包含空格的话，请使用引号括起来。

hexo generate  # 生成静态文件。-d 文件生成后立即部署网站 -w 监视文件变动 
hexo g #简写

hexo publish [layout] <filename>  # 发表草稿。

hexo server
# 启动服务器。默认情况下，访问网址为： http://localhost:4000/。
# -p 重设端口 -s 只使用静态文件 -l 启动日志记录

hexo deploy  # 部署网站。-g 部署之前预先生成静态文件
hexo d #简写

hexo clean  #清除缓存文件 (db.json) 和已生成的静态文件 (public)。

```

  

----



## 6. 使用主题

> `hexo`官方提供了大量的 [优秀主题](https://hexo.io/themes/)，如果选中某款主题，点击主题，移至页面底部，一般都可以看到主题的版权说明，点击即可以进入主题的`github`仓库，如选中第一个主题`A-RSnippet`，下图所示：

![](http://cdn.briarbear.cn/201806221936_116.png)



- 笔者使用的 [yilia](https://github.com/litten/hexo-theme-yilia)主题
- 在`hexo`的目录中，主题文件位于 `./themes`, 主题的主要流程
  - 在主题目录中，`git clone 主题仓库`
  - 修改主题的配置文件：`_config.yml`
- 具体使用流程，可参考一个 [yilla主题使用教程](https://github.com/litten/hexo-theme-yilia) 
- 如果喜欢笔者的主题,已做了一些修改，可以在文本的`github`仓库中`clone`下来，`./themes/yilla`

----

## 7. Markdown编辑器与图床

- 笔者使用的 [Typora](https://typora.io/)编辑器，一款非常优秀的markdown编辑器，满足你对markdown编辑器所有的幻想，（即时预览，仅此一家）
- 在`hexo`中，使用`hexo new`新建文章后，再去`./source/_posts`目录下找到目标文档，使用`Typora`编辑器编辑
- 而使用Markdown编辑器，我们经常会遇到图片问题，这里提供两种处理方案
  - 第一种方案：【相对路径】在`hexo`配置文件中，开启 `post_asset_folder: true`，新建文件会生成文件对应的文件夹，在markdown中可以使用相对路径来引用图片
  - 第二种方案：【绝对路径】笔者使用的方案，使用在线的图床，这样可以更好的方便使用markdown文件，在任意地方打开都能正确的渲染，而使用绝对路径，就得使用图床网站来保存图片
- 关于图床
  - 推荐 [七牛云](https://www.qiniu.com/)，实名认证，送免费存储空间，存储图片够用
- 但是，使用图床，如果我们每次使用一个图片，都要先上传到图床，再复制外链，再粘贴 ，这样效率就低的可怕，有没有一种工具，我复制一张图片，后台自动上传到图床，然后返回外链至剪切板，这样在markdown编辑器中直接就可以使用呢？
- 有，当然有！！！！Mac下有付费的应用，Windows下推荐 [qImage](https://github.com/jiwenxing/qimage-win)，使用方法参看具体链接
- 最终效果：使用 `Alt + S` 截图（推荐截图贴图软件 [snipaste](https://zh.snipaste.com/) ），复制图片，在typora中使用快捷键 `Ctrl + Alt + V`，则直接生成图片

---

## 8. 多端同步与Gitment

- 在`hexo`中，通过`git`连接`github`仓库，它同步的文件实际上仅为经过`hexo`生成的静态文件，那么怎样将源文件在多端同步，这样是可以在多台设备都可以编辑博客，解决思路：`Git 分支`

- 详细的解决方案，[知乎：使用hexo，如果换了电脑怎么更新博客？](https://www.zhihu.com/question/21193762)

- 到此为止，静态博客基本可用，但由于是静态网站，无法评论，解决思路是结合`Gitments`来解决，同时，`yilia`主题也提供了多种评论方式，具体可参看 [wiki](https://github.com/litten/hexo-theme-yilia/wiki/%E5%A4%9A%E8%AF%B4%E3%80%81%E7%95%85%E8%A8%80%E3%80%81%E7%BD%91%E6%98%93%E4%BA%91%E8%B7%9F%E5%B8%96%E3%80%81Disqus%E8%AF%84%E8%AE%BA%E9%85%8D%E7%BD%AE)

- 而使用`Gitments`，[使用教程](https://imsun.net/posts/gitment-introduction/)

- 笔者的配置文件

  ![](http://cdn.briarbear.cn/201806221914_399.png)

  

-----



## 相关链接



- [GitHub+Hexo 搭建个人网站详细教程](https://zhuanlan.zhihu.com/p/26625249)
- [Hexo中文文档](https://hexo.io/zh-cn/docs/)
- [Gitment：使用 GitHub Issues 搭建评论系统](https://imsun.net/posts/gitment-introduction/)
- [知乎：使用hexo，如果换了电脑怎么更新博客？](https://www.zhihu.com/question/21193762)
- [一个简洁优雅的hexo主题 yilia](https://github.com/litten/hexo-theme-yilia)

