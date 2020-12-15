# 基本操作

## 安装Ruby RubyGems

官网下载，直接安装。

## 安装jekyll

gem install jekyll jekyll-paginate bundler rake

## 初始化使用页面

jekyll new ./

jekyll build

jekyll serve

## 坑

bundler的使用会将jekyll build时所需要的所有插件都进行安装，但windows上并不是所有插件都支持。

Rake可以帮助快速创建jekyll规范的md，Rakefile就是具体命令的执行程序，windows支持的不好，[裂开]

## 感谢

感谢hux的<a href="http://huangxuan.me/" target="_blank">theme</a>。