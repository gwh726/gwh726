# 在GitPage上管理docsify生成的技术文档



## 1.拉取技术文档到本地

技术文档分为`团队通用技术文档`和`项目相关技术文档`两种

`团队通用技术文档`目前放置位置为http://git.thunisoft.com/JCW_PRD_FZ_SXJDZHXT/gitpage 后期可能会更变位置

`项目相关技术文档`是每个项目所使用的的相关技术文档，通过项目所在目录即可找到该项目下的子项目`项目名_GitPage`

使用git工具拉取项目代码到本地

## 2.技术文档文件结构

```javascript
 |-- 项目名
    |   |-- .gitlab-ci.yml				//用于提交代码后触发任务更新网页文档内容
    |   |-- package.json
    |   |-- README.md
    |   |-- docs								
    |       |-- .nojekyll				//用于阻止 GitHub Pages 忽略掉下划线开头的文件
    |       |-- index.html				//项目的入口文件
    |       |-- README.md				//显示为主页内容
    |       |-- _coverpage.md			//封面在这里配置
    |       |-- _navbar.md				//顶部菜单栏在这里配置
    |       |-- _sidebar.md				//左侧菜单栏在这里配置
    |       |-- static					//使用的静态文件如引入的JS和图片
    |   |-- .......						//页面引用的.md文档

```

## 3.文档相关配置和修改

文档修改过程中可在本地使用 `docsify-cli` 工具通过指令 `docsify serve docs` 启动一个本地服务器（使用前请使用npm全局安装），访问地址 http://localhost:3000 实时预览效果，以及时发现错误。

### 左侧菜单栏修改

在`_sidebar.md`中可以配置左侧菜单栏信息内容，文本内容格式如下

```
* **环境清单**
* * [环境清单](/环境清单/环境清单.md)
* **技术标准**
* * [IDEA使用sonarlint规范代码](/技术标准/IDEA使用sonarlint规范代码.md)
* **技术问题**
* * [使用node.js接收ActiveMQ遇到的坑](/技术问题/使用node.js接收ActiveMQ遇到的坑.md)
* **技术分享**
* * [Mybatis查询数据慢问题排查](/技术分享/Mybatis查询数据慢问题排查.md)
* **使用文档**
* * [SMD的使用](/使用文档/SMD的使用.md)
* **项目相关文档**
* * [审讯监督指挥系统](http://jcw_prd_fz_sxjdzhxt.page.thunisoft.com/jcw_prd_fz_sxjdzhxt_gitpage/#/)
```

加入新文件就在相应菜单栏下添加内容，如在环境清单栏下添加`其他环境清单`，就在`环境清单`文件夹下加入`其他环境清单.md`文件并修改`_sidebar.md`为

```
* **环境清单**
* * [环境清单](/环境清单/环境清单.md)
* * [其他环境清单](/环境清单/其他环境清单.md)
* **技术标准**
* * [IDEA使用sonarlint规范代码](/技术标准/IDEA使用sonarlint规范代码.md)
* **技术问题**
* * [使用node.js接收ActiveMQ遇到的坑](/技术问题/使用node.js接收ActiveMQ遇到的坑.md)
* **技术分享**
* * [Mybatis查询数据慢问题排查](/技术分享/Mybatis查询数据慢问题排查.md)
* **使用文档**
* * [SMD的使用](/使用文档/SMD的使用.md)
* **项目相关文档**
* * [审讯监督指挥系统](http://jcw_prd_fz_sxjdzhxt.page.thunisoft.com/jcw_prd_fz_sxjdzhxt_gitpage/#/)
```

### 顶部菜单栏修改

在`_navbar.md`中可以配置顶部侧菜单栏信息内容，文本内容格式如下

```
* [首页](/) 
* [关于](/关于/index.md)
```

加入新栏目同左侧菜单栏的添加方式

顶部菜单也可以修改为下拉菜单形式，示例如下

```
* 跳转
  * [关于](/关于/index.md)
  * [起始页](/README.md)
* [关于](/关于/index.md)
```

### 封面修改

在`_coverpage.md`中可以配置封面信息内容，文本内容格式如下

```
# 音视频-讯问指挥团队-通用技术文档
<img src="static/程序出错2.gif"  width="150" />

开发者使用手册，技术文档

常见问题FAQ

[GIT地址](http://git.thunisoft.com/dashboard/projects)
[进入](/README.md)
```

期待强大的大佬提供更加丰富的封面内容

## 4.更新远端的网页文档

修改完成后，提交代码到远端（错误代码将导致网页文档出错），远端将自动更新代码

## 5.使用docsify生成新的技术文档网站

### 本地创建和启动

全局安装 `docsify-cli` 工具，可以方便地创建及在本地预览生成的文档。

```bash
npm i docsify-cli -g
```

进入项目目录，通过 `init` 初始化项目。（已有docs文件请跳过此步）

```bash
docsify init ./docs
```

初始化成功后，可以看到 `./docs` 目录下创建的几个文件

- `index.html` 入口文件
- `README.md` 会做为主页内容渲染
- `.nojekyll` 用于阻止 GitHub Pages 忽略掉下划线开头的文件

直接编辑 `docs/README.md` 就能更新文档内容。

在 `./docs`目录下创建其他md文件（具体内容请参照标题2-3或其他出处）

- `_coverpage.md` 首页文件 （需要在`index.html`中`window.$docsify`内加入`coverpage: true`）
- `_navbar.md` 顶部菜单栏（需要在`index.html`中`window.$docsify`内加入`loadNavbar: true`）
- `_sidebar.md` 左侧菜单栏（需要在`index.html`中`window.$docsify`内加入`loadSidebar: true`）

通过运行 `docsify serve` 启动一个本地服务器，可以方便地实时预览效果。默认访问地址 [http://localhost:3000](http://localhost:3000/) 。

```bash
docsify serve docs
```

### 将本地技术文档提交到GitPage托管

在git上创建一个空项目，拉取代码到本地，将本地技术文档项目拷贝到git项目内，提交代码

等待流水线通过后，以Maintainer或Owner权限登录远端在 设置->Pages就能看到自动生成的技术文档网址

网址命名规则为 http://项目名page.thunisoft.com/子项目名

![](https://files.catbox.moe/m1ndju.png)



