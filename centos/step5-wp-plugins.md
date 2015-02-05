WordPress插件及管理
==================

# 一、修改固定链接
将固定链接格式修改为：
```
/%post_id%.html
```
即显示为`http://www.sitename.com/1234.html`
随后要确保：

> 1. `.htaccess`文件可写
> 2. apache的mod_rewrite模块正常加载（修改`/etc/httpd/conf/httpd.conf`文件）
> 3. Directory的AllowOverride设置为ALL，在/etc/httpd/conf.d/vhost.conf文件中，在自己的站点VirtualHost内加入以下内容：
> ```
> <Directory "/srv/www/bionotes.cn/public">
        options FollowsymLinks Indexes
        AllowOverride All
        Order allow,deny
        Allow from all
     </Directory>
> ```

# 二、常用插件

- Jetpack：包含MarkDown功能。
- Crayon Syntax Highlighter：功能强大的代码高亮插件。
- Akismet ：垃圾评论过滤。
 