
# 个人博客站点 | personal website

预览地址:<br>
	- GitHub: [https://briarbear.github.io](https://briarbear.github.io)
	<br>
	- Gitee: [https://briarbear.gitee.io](https://briarbear.gitee.io)(速度更快)


## 博客搭建笔记
 - [Hexo+GitPage+Gitments+yilia博客搭建手记](https://briarbear.github.io/2018/06/22/Hexo-GitPage-Gitments-yilia%E5%8D%9A%E5%AE%A2%E6%90%AD%E5%BB%BA%E6%89%8B%E8%AE%B0/)

 ---
 ## 说明
 - GitPages默认是根据master分支来显示页面
 - 使用hexo时，是在本仓库的hexo分支下编辑，在该分支下，通过配置文件_config.yml与github的master分支建立关联
 - 然后通过`hexo new title`的方式新建文件，`hexo g`生成文件、`hexo d`部署到github的master分支
 - 而hexo分支下的文件，则是通过正常的git操作完成同步，master分支下的文件则是通过上一行的命令完成了同步
 - 注意同步hexo分支时，是`git push origin hexo`
 - 重新同步该博客：
	- 安装git、node.js、hexo等
	- git clone:`git clone git@github.com:briarbear/briarbear.github.io.git`
	- 在目标文件且hexo分支下执行:`npm install hexo-server --save`
	- 然后再执行其他hexo操作
 - gitee码云上同步该博客
    - 在本地添加远程仓库 `git remote add gitee git@gitee.com:briarbear/briarbear.git`
	- 执行`git add -A`,`git commit -m "**"`,`git push gitee hexo`
	- 然后在gitee页面服务里的GiteePages选择部署分支，这里选择`hexo`分支即可