# 概述

目前有众多的知识管理软件，对于比较私密的文档使用印象笔记，不仅支持Markdown这种轻量级的标记语言，也支持复杂的富文本语言，但是一些比较有用的功能则需要收费，比如查看历史版本、离线访问、容量限制。最近，使用了一下阿里的语雀，书写体验、排版和易操作方面确实属于不错的一款产品，可是，某一天断网了，不支持离线访问是硬伤。因此，想要寻找如下一款产品满足如下需求：

- 沉浸式的书写体验，不需要丰富的社交场景，也不需要过于酷炫的功能，只想安安静静的码字。
- 离线编辑，想什么时候访问自己做主。同时，同步是必不可少的。
- 支持版本管理。
- 不收费，也没有容量限制。想写多少写多少。
- 支持私有笔记。
- 支持同其他同事、同学沟通，因为大部分都是程序员，所以可以查看Markdown文件是必不可少的。
- 对笔记中的目录结构没有限制。
- 对笔记中关联的图像，本地和远程都可以访问，而且没有容量限制。
- 可以方便的导出其他格式，比如html，PDF等。

通过调研，使用GitHub&GitBook&Typora方案，可以完美的解决以上问题。



#  搭建环境

**GitHub**

- GitHub创建develop-stack项目，其中.gitignore选择gitbook
- 回到本地创建本地目录 `mkdir note && cd note`
- 下载项目到本地 `git clone https://github.com/sciatta/develop-stack.git`

**GitBook**

- 安装nodejs `brew install node`
- 安装GitBook命令行工具 `npm install -g gitbook-cli`
- 在develop-stack目录执行 `gitbook init` 初始化Gitbook目录环境，生成两个重要文件README.md和SUMMARY.md

**Typora**

- 编辑SUMMARY.md描述项目的目录和文件结构
- 在develop-stack目录执行 `gitbook init` 。GitBook会查找SUMMARY.md文件中描述的目录和文件，如果没有则会将其创建。<font color=red>注意，如果删除SUMMARY.md描述项目的目录和文件结构，执行 gitbook init 命令不会删除相应的目录或文件，需要手动维护</font>。

**GitBook**

* 本地运行 `gitbook serve --port 8088` 开启GitBook服务。通过 http://localhost:8080 访问本地服务。在执行命令的同时会执行构建命令 `gitbook build` 生成 `_book` 目录。<font color=red>注意， `_book` 目录是临时目录，每次构建时全部重建</font>。如果退出服务的话，执行 `control + c` 即可
* 本地构建 `gitbook build ./ docs` 。

**GitHub**

- 执行 git add 命令 `git add -A` 将文件提交到暂存区
- 执行 git commit 命令 `git commit -m “test”` 将文件提交到本地仓库
- 执行 git push 命令 `git push origin master` 将文件提交到远程仓库
- GitHub的develop-stack项目，Settings标签页的的GitHub Pages，修改Source选择 `master branch/docs folder` 。成功后显示 Your site is ready to be published at https://sciatta.github.io/develop-stack/ ，稍后即可访问GitHub Pages网站。



# GitBook插件

在develop-stack目录创建book.json文件，配置插件后，运行 `gitbook install` 命令自动安装插件。

##  hide-element

隐藏默认gitbook左侧提示：Published with GitBook

```json
{
    "plugins": [
        "hide-element"
    ],
    "pluginsConfig": {
        "hide-element": {
            "elements": [".gitbook-link"]
        }
    }
}
```

## expandable-chapters

gitbook默认目录没有折叠效果。

```json
{
    "plugins": [ "expandable-chapters" ]
}
```

## code

在代码区域的右上角添加一个复制按钮，点击一键复制代码。

```json
{
    "plugins" : [ "code" ]
}
```

## splitter

左侧目录和右侧文章可以拖动调节宽度。

```json
{
    "plugins": [
        "splitter"
    ]
}
```

## search-pro

支持中英文。

```json
{
    "plugins": [
          "-lunr", 
          "-search", 
          "search-pro"
    ]
}
```

##pageview-count

记录每个文章页面被访问的次数。本质是访问 https://hitcounter.pythonanywhere.com/

```json
{
  "plugins": [ "pageview-count"]
}
```

## tbfed-pagefooter

在每个文章下面标注版权信息和文章时间。

```json
{
    "plugins": [
       "tbfed-pagefooter"
    ],
    "pluginsConfig": {
        "tbfed-pagefooter": {
            "copyright":"Copyright &copy sciatta.com 2020",
            "modify_label": "修订时间: ",
            "modify_format": "YYYY-MM-DD HH:mm:ss"
        }
    }
}
```

## popup

点击可以在新窗口展示图片。

```json
{
  "plugins": [ "popup" ]
}
```

## auto-scroll-table

为避免表格过宽，增加滚动条。

```json
{
  "plugins": ["auto-scroll-table"]
}
```

## -sharing

去掉 GitBook 默认的分享功能。由于默认的一些推特，脸书都需要翻墙，所以将分享功能全部关闭。

```json
"plugins": [
	"-sharing"
]
```

## github-buttons

给 GitBook 添加 GitHub 的图标来显示 star 和 follow。

```json
{
  "plugins": [
    "github-buttons"
  ],
  "pluginsConfig": {
	"github-buttons": {
      		"buttons": [{
        		"user": "sciatta",
        		"repo": "develop-stack",
        		"type": "star",
        		"count": true,
        		"size": "small"
      		}]
    }
  }
}
```

## github

在右上角显示 github 仓库的图标链接

```json
{
    "plugins": [ "github" ],
    "pluginsConfig": {
        "github": {
            "url": "https://github.com/sciatta"
        }
    }
}
```

## Google统计

添加Google统计

Google统计 https://analytics.google.com/ 在一个平台上可以全面分析业务数据，进而做出更明智的决策。在网站注册，获取跟踪ID 

```json
{
    "plugins": ["ga"],
    "pluginsConfig": {
        "ga": {
            "token": "UA-156491400-1"
        }
    }
}
```

## BaiDu

添加BaiDu统计

百度统计https://tongji.baidu.com/。在网站注册，获取跟踪ID 

```json
{
    "plugin": ["baidu"],
    "pluginsConfig": {
        "baidu": {
            "token": "c6612709c010da681bbd4b785968a638"
        }
    }
}
```

## Donate

Gitbook 捐赠打赏插件

```json
{
    "plugins": ["donate"],
    "pluginsConfig": {
        "donate": {
            "wechat": "/assets/wechat.jpg",
            "alipay": "/assets/alipay.jpg",
            "title": "",
            "button": "捐赠",
            "alipayText": " ",
            "wechatText": " "
        }
    }
}
```

## anchors

标题带有 github 样式的锚点

```json
{
	"plugins" : [ "anchors" ]
}
```

## anchor-navigation-ex 

页面内导航，一键回到顶部。

```json
{
    "plugins": ["anchor-navigation-ex"],
	"pluginsConfig": {
        "anchor-navigation-ex": {
            "showLevel": true,
            "associatedWithSummary": false,
            "printLog": false,
            "multipleH1": true,
            "mode": "float",
            "showGoTop": true,
            "float": {
                "floatIcon": "fa fa-navicon",
                "showLevelIcon": false,
                "level1Icon": "fa fa-hand-o-right",
                "level2Icon": "fa fa-hand-o-right",
                "level3Icon": "fa fa-hand-o-right"
            },
            "pageTop": {
                "showLevelIcon": false,
                "level1Icon": "fa fa-hand-o-right",
                "level2Icon": "fa fa-hand-o-right",
                "level3Icon": "fa fa-hand-o-right"
            }
		}
    }
}
```

## sitemap

生成站点地图，便于爬虫抓取页面。可以通过http://www.sciatta.com/sitemap.xml 访问。

```json
{
    "plugins": ["sitemap"],
    "pluginsConfig": {
        "sitemap": {
            "hostname": "http://www.sciatta.com/"
        }
    }
}
```



# Typora设置

偏好设置 | 编辑器 | 图片插入

- 复制图片到 ./${filename}.assets 文件夹

- 优先使用相对路径



偏好设置 | 通用 | 启动选项 | 打开指定目录

- 选择工作目录



# 绑定域名

- 获取GitHub Pages的IP地址

```bash
ping -c 3 sciatta.github.io
```

<font color=red>注意，ping的时候不需要加仓库的名称</font>。

- 配置阿里云

进入阿里云解析列表，添加记录：

| 记录类型 | 主机记录 | 记录值          |
| -------- | -------- | --------------- |
| A        | @        | 185.199.111.153 |
| A        | www      | 185.199.111.153 |

- 配置GitHub Pages

GitHub的develop-stack项目，Settings标签页的的GitHub Pages，修改Custom domain：www.sciatta.com