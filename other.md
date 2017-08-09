**使用本文方法安装的程序版本:**
> Nginx 1.10.0  
> PHP7.0.6 (PHP-FPM) / PHP5.6.12(PHP-FPM)  
> MariaDB 10.1.13  

## 添加`Nginx`源  

执行命令,安装nginx yum源  
```bash
rpm -ivh http://nginx.org/packages/centos/7/noarch/RPMS/nginx-release-centos-7-0.el7.ngx.noarch.rpm 
```


## 添加`PHP5.6/PHP7`源  
PHP的源使用 `php.net` 官方推荐源: [iuscommunity(ius)](https://iuscommunity.org)  
请直接执行下面命令安装rpm  
```bash
rpm -ivh https://dl.fedoraproject.org/pub/epel/7/x86_64/e/epel-release-7-7.noarch.rpm

rpm -ivh https://dl.iuscommunity.org/pub/ius/stable/CentOS/7/x86_64/ius-release-1.0-14.ius.centos7.noarch.rpm
```

## 添加`MariaDB`源
[MariaDB](http://mariadb.org)的yum源使用官方提供的源地址
官方没提供rpm来直接导入,需要手动创建repo文件

```bash
vi /etc/yum.repos.d/MariaDB.repo
```
### 插入repo文件以下内容（一个字符都不能少）  
```vim
# MariaDB 10.1 CentOS repository list - created 2015-11-12 02:20 UTC
# http://mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.1/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
```

## 执行安装

### PHP5.6.X 版本
```bash
yum install -y MariaDB-server MariaDB-client nginx php56u-fpm php56u-mbstring php56u-bcmath php56u-mcrypt php56u-xmlrpc php56u-pdo php56u-xml php56u-xmlrpc php56u-mysqlnd php56u-gd php56u-opcache
```

### PHP7.0.X 版本
```bash
yum install -y MariaDB-server MariaDB-client nginx php70u-fpm php70u-gd php70u-json php70u-intl php70u-mbstring php70u-mcrypt php70u-mysqlnd php70u-opcache php70u-pdo php70u-pdo-dblib php70u-process php70u-pgsql php70u-recode php70u-xml php70u-xmlrpc php70u-cli
```

## 启动 nginx/php/mysql
```bash
systemctl start php-fpm
systemctl start nginx
systemctl start mariadb
```

### 开机自启服务
```bash
systemctl enable nginx
systemctl enable php-fpm
systemctl enable mariadb
```

### 设置默认MySQL/MariaDB密码

```bash
mysqladmin -u root password 'new-password'
```
请将 `new-password` 替换为你的数据库密码

### 创建 `Shadowsocks-Panel` nginx conf文件
```bash
vi /etc/nginx/conf.d/yourdomain.com.conf
```
写入以下内容：

```nginx
server{
    listen 80;
    server_name yourdomain.com;
    access_log /home/wwwlogs/yourdomain.com_nginx.log combined;
    index index.html index.htm index.php;
    root /home/wwwroot/shadowsocks-panel/Public;

    if (!-e $request_filename) {
        rewrite (.*) /index.php last;
    }

    location ~ .*\.(php|php5)?$ {
        fastcgi_split_path_info ^(.+?\.php)(/.*)$;
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}

```
其中参数 `root` , `server_name`, `access_log` 需要自行修改到你程序文件位置。(`access_log`需要读写权限,`root`需要读取权限)

```bash
# 重启 Nginx
systemctl restart nginx
```

### 创建MYSQL数据库
```bash
$ mysql -uroot -p
# 输入你的设定的数据库密码，进入MYSQL命令行模式之后执行下面这句创建库，请忽略 > 号
> create database shadowsocks;
```

**其他设置按照面板的安装教程来**

### 其他问题
> 如果需要使用别的php支持库，可以直接使用`yum search php56u`可以得到所有可用支持库列表,挑选需要的安装即可.
> PHP7查找安装php扩展(支持库) `yum search php7` 即可搜索到全部支持库  
> 照样使用正常的 `yum install php7.0-pdo` 安装完毕 `systemctl restart php-fpm` 扩展即可生效。非常方便，完全不需要重新编译这类麻烦事情。
