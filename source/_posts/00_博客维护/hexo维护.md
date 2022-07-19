---
title: Hexo博客维护
date: 2022-07-07 14:10:52
categories:
- 博客维护
tags:
---

## 1 Hexo博客维护

版本：`Next 8.12.2`

1、下载安装nodejs

https://nodejs.org/en/

2、切换npm为阿里源

```shell
 npm config set registry https://registry.npm.taobao.org
```

3、使用npm安装hexo

```shell
npm install -g hexo-cli
```

>否则会报错:
>
>$ hexo clean
>ERROR Cannot find module 'hexo' from 'C:\Users\fr724\Desktop\新建文件夹\gsproj.github.io'
>ERROR Local hexo loading failed in ~\Desktop\新建文件夹\gsproj.github.io
>ERROR Try running: 'rm -rf node_modules && npm install --force'

4、使用npm安装搜索功能依赖

```shell

npm install hexo-excerpt --save
```

5、内容上线

```shell
hexo clean
hexo g
hexo d # 上线到githubio
hexo s # 本地试运行
```



## 2 功能添加

### 2.1 搜索功能

安装依赖

```shell
npm install hexo-generator-searchdb --save
```

修改主题设置文件`themes/next/_config.yml`

```yml
# Local Search
# Dependencies: https://github.com/next-theme/hexo-generator-searchdb
local_search:
  enable: true	# 此处改为true
  # If auto, trigger search by changing input.
  # If manual, trigger search by pressing enter key or search button.
  trigger: auto
  # Show top n results per article, show all results by setting to -1
  top_n_per_article: 1
  # Unescape html strings to the readable one.
  unescape: false
  # Preload the search data when the page loads.
  preload: false
```

### 2.2 菜单显示归档/书签

修改主题设置文件next/_config.yml

```yaml
menu:
  #home: / || fa fa-home
  #about: /about/ || fa fa-user
  tags: /tags/ || fa fa-tags	# 取消注释
  categories: /categories/ || fa fa-th	# 取消注释
  archives: /archives/ || fa fa-archive	# 取消注释
  #schedule: /schedule/ || fa fa-calendar
  #sitemap: /sitemap.xml || fa fa-sitemap
  #commonweal: /404/ || fa fa-heartbeat
```

效果如下图：

![image-20220719101836015](../../img/image-20220719101836015.png)

### 2.3 鼠标点击烟花效果

下载[fireworks.js](https://aliyun-oss-pic-bucket.oss-cn-beijing.aliyuncs.com/file/fireworks.js)文件放到`themes/next/source\js\cursor`中

创建`themes/next/layout/_custom`文件夹，并在其中创建`custom.swig`文件，内容如下

```python
{# 鼠标点击烟花特效 #}
{% if theme.cursor_effect == "fireworks" %}
  <script async src="/js/cursor/fireworks.js"></script>
{% elseif theme.cursor_effect == "explosion" %}
  <canvas class="fireworks" style="position: fixed;left: 0;top: 0;z-index: 1; pointer-events: none;" ></canvas>
  <script src="//cdn.bootcss.com/animejs/2.2.0/anime.min.js"></script>
  <script async src="/js/cursor/explosion.min.js"></script>
{% elseif theme.cursor_effect == "love" %}
  <script async src="/js/cursor/love.min.js"></script>
{% elseif theme.cursor_effect == "text" %}
  <script async src="/js/cursor/text.js"></script>
{% endif %}
```

编辑`themes/next/layout/_layout.njk文件，在尾行导入swig文件

```shell
...
  {{ partial('_scripts/index.njk', {}, {cache: theme.cache.enable}) }}
  {{ partial('_third-party/index.njk', {}, {cache: theme.cache.enable}) }}
  {{ partial('_third-party/statistics/index.njk', {}, {cache: theme.cache.enable}) }}

  {%- include '_third-party/math/index.njk' -%}
  {%- include '_third-party/quicklink.njk' -%}

  {{- next_inject('bodyEnd') }}
  {% include '_custom/custom.swig' %}	# 导入swig文件
</body>
</html>
```

编辑`themes/next/_config.yml`文件，添加

```yaml
# 鼠标点击烟花特效
cursor_effect: fireworks
```

> **PS:一个小修改，防止烟花残留**
>
> 参考：https://yfx2012.top/2022/01/17/hexo/mouse-click-fireworks/
>
> 修改firewaors.js文件
>
> ```javascript
>   handlePageHide() {
>     this.booms = []
>     this.running = false
>     // 简单修改，清理停留不动的烟火特效
>     this.computerContext.clearRect(0, 0, this.globalWidth, this.globalHeight)
>     this.renderContext.clearRect(0, 0, this.globalWidth, this.globalHeight)
>   }
> ```

### 2.4 关闭文章目录的自动章节号

未修改前，自动排了序号，看上去很乱：

![image-20220719104324299](../../img/image-20220719104324299.png)



修改主题配置文件`/themes/next/_config.yml`

```yaml
toc:
  enable: true
  # Automatically add list number to toc.
  number: false	# 改为false关闭
  # If true, all words will placed on next lines if header width longer then sidebar width.
  wrap: false
  # If true, all level of TOC in a post will be displayed, rather than the activated part of it.
  expand_all: false
  # Maximum heading depth of generated toc.
  max_depth: 6
```

修改后，清爽了很多：

![image-20220719104519550](../../img/image-20220719104519550.png)

### 2.5 添加头像

将头像文件放到`themes/next/source/images`中

修改主题配置文件`/themes/next/_config.yml`

```shell
# Sidebar Avatar
avatar:
  # Replace the default image and set the url here.
  url: /images/myavatar.png
```

### 2.6 修改背景图片

>白花花的有点单调

将背景图片放到`themes\next\source\images`

新建`themes\next\source\_data`文件夹

在`_data`中新建`styles.styl`文件

```yaml
body {
    background:url(/images/xhc.jpg);
    background-repeat: no-repeat;
    background-attachment:fixed;
    background-position:50% 50%;
    // background-size: 100% 100%;
    background-size: cover;
    -webkit-background-size: cover;
    -o-background-size: cover;
    -moz-background-size: cover;
    -ms-background-size: cover;
}
```

然后将`themes/next/_config.yml`配置文件中`custom_file_path:`下的`#style: source/_data/styles.styl`#号去掉，如下

```yaml
custom_file_path:
  #head: source/_data/head.njk
  #header: source/_data/header.njk
  #sidebar: source/_data/sidebar.njk
  #postMeta: source/_data/post-meta.njk
  #postBodyEnd: source/_data/post-body-end.njk
  #footer: source/_data/footer.njk
  #bodyEnd: source/_data/body-end.njk
  #variable: source/_data/variables.styl
  #mixin: source/_data/mixins.styl
  style: source/_data/styles.styl
```

### 2.7 修改为圆角

根据当前使用的主题，编辑主题对应的style文件，如`\themes\next\source\css\_variables\Gemini.styl`

在里面添加：

```css
// 圆角设置
$border-radius-inner     = 20px;
$border-radius           = 20px;
```

### 2.8 加载动画速度调整

页面加载会有段动画，默认比较慢，可以调整

编辑文件`themes\next\source\js\motion.js`，调整`duration`的值，默认200，越大越慢，可以调小一些

```css
bootstrap: function() {
    if (!CONFIG.motion.async) this.queue = [this.queue];
    this.queue.forEach(sequence => {
        const timeline = window.anime.timeline({
            duration: 100,	# 调整为100
                easing  : 'linear'
        });
        sequence.forEach(item => {
            if (item.deltaT) timeline.add(item, item.deltaT);
            else timeline.add(item);
        });
    });
}
```



