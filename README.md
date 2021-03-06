# 基本操作

## 安装Ruby RubyGems

官网下载，直接安装。

## 安装jekyll

gem install jekyll jekyll-paginate bundler rake

## 初始化使用页面

```bash
jekyll new ./

jekyll build

jekyll serve
```

## mermaid安装及用法

> 已集成

项目地址：https://github.com/jasonbellamy/jekyll-mermaid

1. 安装 本地&github增加action
   
```bash
gem install jekyll-mermaid
```

2. 添加`_config.yml`支持

```yaml
gems: [jekyll-mermaid]
```

3. 设置，js需要自己下载，使用版本，8.8.4

https://unpkg.com/mermaid@8.8.4/dist/mermaid.js
https://unpkg.com/mermaid@8.8.4/dist/mermaid.js.map
https://unpkg.com/mermaid@8.8.4/dist/mermaid.min.js
https://unpkg.com/mermaid@8.8.4/dist/mermaid.min.js.map

```yaml
mermaid:
  src: '/js/mermaid.js'
```

4. 简单例子

```
{% mermaid %}
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
{% endmermaid %}
```

使用方法：https://mermaid-js.github.io/mermaid/

## 坑

* bundler的使用会将jekyll build时所需要的所有插件都进行安装，但windows上并不是所有插件都支持。

* Rake可以帮助快速创建jekyll规范的md，Rakefile就是具体命令的执行程序，windows支持的不好[裂开]。

* mermaid复杂的图表画出来无法定位，只能确定关系正确，还是换个画图工具吧。

## 感谢

感谢hux的<a href="http://huangxuan.me/" target="_blank">theme</a>。

## DONE

1. github action 自动化流水线--->阿里云OSS
2. 站长统计
3. https 支持
4. CDN 支持
5. tags fixed
6. mermaid  流程图插件，流水线优化


## TODO

1. RSS
2. 浏览量
3. 转发
4. 打赏:)

