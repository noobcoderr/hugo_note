## 使用Hugo + Github Page构建自己的个人博客

1. 安装hugo

   macos: brew install hugo

   windows: 源码安装后记得将可执行文件加如到环境变量中

2. 使用hugo创建网站

   hugo new site blog

3. 生成文章

   hugo new posts/first-post.md

4. 加载主题

   git init 

   git submodule 主题git themes/主题名

5. 修改配置文件

   ```shell
   echo 'theme = "主题名"' >> config.toml
   ```

6. 本地构建

   hugo server

   即可在http:127.0.0.1:1313/ 查看到构建后的页面了

7. 构建页面

   hugo 

   即可将网页生成到默认的public子目录里。当然你可以指定一个别的文件夹

   ```shell
   echo 'publishDir = "docs"' >> config.toml
   ```

   即以后生成的网页静态文件都到docs子目录下了。

8. 发布到github.io

   将生成在docs下的网页文件推送到 `yourname.github.io`库下，稍等一会儿你就可以看到你生成的网页了。



问题：

以上方法是得写好文章，然后将构建好的静态文件更新到github.io库下，即每次都得构建一次，是不是可以有一种，只用将文章写好后，推送到github.io上即可自动化构建然后推送呢？持续交付？