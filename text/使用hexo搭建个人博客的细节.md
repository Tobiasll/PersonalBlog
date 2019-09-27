---
title: 使用hexo搭建个人博客的细节
date: 2019-09-26 14:05:45序言 
---

一直都想写点技术文章，但由于鄙人不才，知识储备太少，所以工作一年都是看书和看视频等知识输入类型，并没有进行知识输出，现在决定搭建个人博客，不断学习的过程中写点东西，来当作自己的总结和分享



# 博客开源框架选择

搭建博客有很多种方式，可以完全靠自己纯手工搭建，或采用开源的东西，前者本人觉得太折腾，所以不太适合我这种只想写点文章的个人用户，所以我选了开源静态博客框架加GitHub Page来托管等实现方案，开源框架目前比较火的有Hexo 、Jekyll、WordPress、ghost、Simple，Octopress...

	## Hexo 和 Jekyll

这是两种我比较了解的框架，两个都是比较主流的选择下面说一下选择原因吧！

1. Jekyll的底层是 Ruby，而Hexo 的底层是NodeJS，所以要求你有NodeJS环境或者Ruby，由于本人上班的电脑是Windos环境，安装Ruby感觉会比NodeJS麻烦，所以这一点我还是选了Hexo

2. 虽然两者都是支持Markdown语法和有很多主题，但是我更喜欢Hexo的主题

3. 之前的蚂蚁金服的sofa stack群里发现他们的sofa官网是Hexo写的，同时也期待他们开源这套主题模板，本人想用:joy:

   ![1569480184234](/blog_image/text1/1569480184234.png)

   废话不多说，下面开始安装搭建环境，hexo官网:<https://hexo.io/>

# Github创建项目

由于我们并没有后端服务器，所以我们把博客的东西都放到GitHub上，让GitHub做托管，这里由于我没有申请域名所以我还是用标准的GitHub Page ，由于GitHub有个特点就是它会用**你的用户名.github.io/项目名/**访问你仓库的index.html，
例如：tobiasll.github.io/HappyBirthday
所以我们要创建一个项目名叫 **你的GitHub用户名.github.io**来作为你的blog仓库，到时候如何没有买域名的小伙伴就可以直接用你的用户名.github.io不用加上项目名就可以直接访问你的博客了

## 新建项目

首先到github创建一个项目，项目名为你的用户名.github.io

![1569481199824](/blog_image/text1/1569481199824.png)

为什么图片要用红圈圈着SSH，因为后面我们可以用SSH keys 方式部署hexo提交代码，就不用每次都输入用户名和密码

## 安装Git

既然是托管到github，那肯定要安装Git啊！Git官网:<https://git-scm.com/>，git的安装自行Google

安装完我们要设置ssh key  ，首先进入或者在菜单里搜索 Git Bash，设置 user.name 和 user.email 配置信息：

```commonlisp
git config --global user.name "你的GitHub用户名"
git config --global user.email "你的GitHub注册邮箱"
```

然后生成设置SSH密钥文件

```commonlisp
ssh-keygen -t rsa -C "你的GitHub注册邮箱"
```

![1569482989245](/blog_image/text1/1569482989245.png)

输入命令后去C盘框生存目录的地址找到id_rsa.pub文件然后复制这个文件的密钥字符串到github 的新增ssh key中

![1569483307400](/blog_image/text1/1569483307400.png)

然后到gitBash 测试一下输入

```commonlisp
 ssh git@github.com
```

![1569483634365](/blog_image/text1/1569483634365.png)

# 安装NodeJS

Hexo 基于 Node.js，Node.js 下载地址：[Download | Node.js](https://link.zhihu.com/?target=https%3A//nodejs.org/en/download/) 下载安装包，注意安装 Node.js 会包含环境变量及 npm 的安装，安装后，检测 Node.js 是否安装成功，在命令行中输入 node -v

检测 npm 是否安装成功，在命令行中输入 npm -v :

# 安装Hexo

先到你的想要存放的hexo数据的目录下创建一个空的文件夹，然后进去文件夹，在地址栏输入cmd进入命令行![1569483903636](/blog_image/text1/1569483903636.png)

然后输入安装hexo的命令

```commonlisp
npm install -g hexo-cli 
```

安装完成输入,初始化我们的博客

```commonlisp
hexo init blog
```

这是初始化完成就会出现一个blog文件夹

![1569484086387](/blog_image/text1/1569484086387.png)

然后在命令行测试hexo是否搭建成功，输入(注意如掉//后面的内容，)

```commonlisp
hexo n test_tobias      # 注释 hexo new "我的博客" 新建文章
hexo g                  # hexo generate  生成
hexo s  				# hexo server 启动服务预览
```

![1569484661570](/blog_image/text1/1569484661570.png)

然后在浏览器输入：http://localhost:4000，出现以下页面则说明搭建成功

![1569484704075](/blog_image/text1/1569484704075.png)

# 推送本地hexo到github托管

因为我们要做到的时让别人能通过外网访问我们的博客，上面的方式是在本机部署的方式，所以我们要把代码放到GitHub上，让github24小时托管，我们每次在本地写完文章或修改配置后部署到github上就好了

为了达到此目的，我们需要修改_config.yml配置文件，绑定我们的git地址，但是_config.yml是有两个一模一样的名子的，希望大家注意一下，别被坑到

1. 第一个是在blog文件夹下的，是用来配置hexo站点的**站点**配置文件

   ![1569485334754](/blog_image/text1/1569485334754.png)

2. 第二个是在themes文件下的，是用来配置主题也就是页面样式的**主题**配置文件

   ![1569485426158](/blog_image/text1/1569485426158.png)

那我们需要推送网站到github上，所以我们就需要配置站点的git地址，所以我们要修改**blog目录**下_config.yml的**站点**配置文件

打开config.yml文件添加 下面的配置：type、repo、brach前面一定要有两个空格，冒号：后面也要一个空格

```yml
deploy:
  type: git
  repo: 这里填入你之前在GitHub上创建仓库的ssh完整路径，记得加上.git
  branch: master
```

![1569485764012](/blog_image/text1/1569485764012.png)

注意repo地址是ssh地址，不是https地址，

![1569485961666](/blog_image/text1/1569485961666.png)

然后按保存，其实就是给 hexo d 这个命令做相应的配置，让 hexo 知道你要把 blog 部署在哪个位置，很显然，我们部署在我们 GitHub 的仓库里。最后安装 Git 部署插件，输入命令：

```commonlisp
npm install hexo-deployer-git --save
```

安装完就可以开始推送博客了

``` commonlisp
hexo clean  # 清除缓存文件 (db.json) 和已生成的静态文件 (public)。
在某些情况（尤其是更换主题后），如果发现您对站点的更改无论如何也不生效，您可能需要运行该命令。
hexo g # hexo generate 生成静态文件。
hexo d # hexo deploy 部署网站。
```

然后出现，这时你的GitHub项目也有一堆文件![1569486164154](/blog_image/text1/1569486164154.png)

然后就可以在你的浏览器输入 你的github用户名.github.io就可以访问博客了，ok，这时候如果成功出现页面则说明推送成功了，也可以正常使用，只是页面有很多参数都不是你想要的，而且很丑，所以这时候我们要去修改参数和主题，使这个博客变成真正属于你的网站

# 修改站点参数

站点的config.yml可以修改一部分参数，例如

![1569486666588](/blog_image/text1/1569486666588.png)

![1569486739448](/blog_image/text1/1569486739448.png)

但是修改内容非常有限，所以这里不建议修改站点配置来达到目的，我们去修改themes下的另一个config.yml文件来达到修改样式的目的，那不喜欢这个主题的样式怎么办，这时候我们就可以换个主题，hexo是一个支持自定义主题模板的框架，同时官网也有很多开源的主题开源更换

# 更换主题

![1569487354825](/blog_image/text1/1569487354825.png)

到官网选一个自己喜欢的主题，然后点图片进去预览，点红框的主题模板名字就可以进去模板的github了，里面都有很详细的配置和安装教程

![1569487435408](/blog_image/text1/1569487435408.png)

然后就一步步按着安装就好了，然后遇到坑就自己去Google解决

# archer主题启用 Algolia 搜索

因为我用的模板是支持Algolia服务的，里面有个小坑，安装步骤

主题启用Algolia的链接<https://github.com/fi3ework/hexo-theme-archer/wiki/%E5%90%AF%E7%94%A8-Algolia-%E6%90%9C%E7%B4%A2>

![1569487753234](/blog_image/text1/1569487753234.png)

其中这一段是有坑的，因为cmd打开的命令行执行hexo aligolia一直报错![1569487913848](/blog_image/text1/1569487913848.png)

加上当时上班中午没睡觉在搭博客，有点懵逼，不断在从新添加apikey和换不同的apiKey来尝试，但是都没生效，于是我就在不用set 变量了改用git bash的export了，结果一次就成功了，所以这个坑，如果大家遇到的话，注意一下

# 使用Hexo

其实一般正常使用就记住那几个最简单的命令，也可以上官网看看命令，

1.  ```commonlisp
   hexo new [layout] <title>
   新建一篇文章。如果没有设置 layout 的话，默认使用 _config.yml 中的 default_layout 参数代替。如果标题包含空格的话，请使用引号括起来。
   特别想开启about me功能时，执行 hexo new page "about"
   在 hexo 目录下 source/about/index.md 中添加字段 layout: about（这个字段必须有且不可更改为其他）
   不能用 hexo n "文章名"
   hexo n 新增的文章会保存在\hexo\blog\source\_posts文件夹下，直接去哪里打开md文件编写内容保存即可
   hexo new page "文章名" 出现在\hexo\blog\文章名\aboutindex.md 
    ```

2.  ```commonlisp
   hexo generate
   生成静态文件。
   简写 hexo g
   编写完文章，运行hexo g就好生存新增静态文件，然后部署到GitHub即可
    ```

3. ```commonlisp
   hexo deploy
   部署网站。
   简写 hexo d
   ```

4. ```
   hexo clean
   清除缓存文件 (db.json) 和已生成的静态文件 (public)。
   
   在某些情况（尤其是更换主题后），如果发现您对站点的更改无论如何也不生效，您可能需要运行该命令。
   ```

5. other 看官网，或者等更新，常用的就这几个

# 小结

其实搭建博客基本一步步按照说明搭建，坑会很少的，而且很快就搭建起来的，关于hexo的使用遇到小坑或好用的骚操作，由于自己也是第一次用这个东西，加上鄙人不才，所以就写了一点点，后续我再不断更新博客，希望大家搭建博客都非常顺利，第一次写博客，花了我两个小时，写的非常不好，希望大家见谅，文章有错误和需要修改的地方，或者有好的建议都可以在博客判断的微信二维码或者邮箱或者GitHub issue 给我发信息，谢谢大家！

关于**绑定域名**这块请看下面《[GitHub+Hexo 搭建个人网站详细教程](<https://zhuanlan.zhihu.com/p/26625249>)》的文章，由于我并没有操作过，以及我没有需要补充的地方，所以我就没有写这一块的内容

# 参考资料

《[GitHub+Hexo 搭建个人网站详细教程](<https://zhuanlan.zhihu.com/p/26625249>)》

《[Hexo 官网](hexo.io)》