## 操作目录
- cd 
- ll    ：ls -l命令的简写
- ls -l  :文件夹下的文件列表
- ls -a  ：显示全部文件
- pwd   ：当前目录

## 文件操作
- mkdir   创建文件夹，参数-p
- rm -r -f  删除，严重防范
- touch  新建文件
- vi 1.txt  （如果文件存在就是编辑，不存在就是创建）
- vi 
  - i: 进入编辑状态
  - esc:退出编辑状态
       （dd、p、yy 都是在esc后可操作）
      - dd : 在退出编辑状态下，删除当前光标所在的一行（也具有剪切功能）
      - p :粘贴（剪切后粘贴）行
      - yy : 复制 行
  - :wq :保存并退出
  - :q! :不保存当前修改并退出

- cp  复制
- mv  剪切
- cat 查看文件内容


## 获取主机名 hostname

## 用户、组操作

- whoami  显示当前用户
- who     显示已经登录的用户
- w      显示已经登录的用户的操作行为

- useradd username  创建用户
- id username  获取用户的详情信息
- usermod -l newname oldname 修改用户名
- usermod -u uid name   修改用户id
- usermod -g gid name  修改用户组id
还有其他参数 
-d    指定用户目录
-s    指定登录Shell



- userdel username 删除用户 （会保留home下的用户目录）
- userdel -r username 删除用户（连同home下的用户目录一起删除）


- groupadd name 添加组
- groupmod -n newname oldname 更改组名
- groupmod -g newid name 更改组id
- groupdel name 删除组

## 修改权限
- chmod： 
    参数 u、g、o 分别是用户，用户组，其他组
        a 代表ugo
        +、- 代表增加或删除权限
        r、w、x 分别是读写执行权限
        -R 递归更改

    实例 ： chmod o+wx 1.txt  给1.txt文件添加w 、 x 权限

- chgrp 改变文件的所属组  chgrp groupname filename         参数-R 递归更改
- chown 改变文件的所属用户  chown username filename        参数-R 递归更改


## yum 基本命令
  - yum install package-name 安装包
  - yum remove package-name 卸载安装包
  - yum update package-name 更新安装包
  - yum info pageage-name 查看安装包信息
  - yum list 查看所有安装包列表
  - yum search keywords 根据关键字查找



##  ---------------------------------------------linux 下使nginx---------------------------------------
#### 1.安装依赖模块
- yum -y install gcc(编译c语言) gcc-c++（编译c++） autoconf（自动配置） pcre pcre-devel（自动匹配） make automake（自动编译）
- yum -y install wget（用于下载文件） heepd-tools(实现http服务) vim（编辑器）


#### 2. CentOS下YUM安装nginx
- vi /etc/yum.repos.d/nginx.repo  创建安装nginx的配置文件
- 进入文件，写入内容
   - [nginx]
     name=nginx repo
     baseurl=http://nginx.org/packages/centos/7/$basearch/
     gpgcheck=0
     enabled=1
- yum -y install nginx  安装

- nginx -v 查看版本（v小写）
- nginx -V 查看配置参数（V大写）
- rpm -ql nginx 查看nginx 的安装目录
- cd /etc/nginx 查看配置目录

- vi nginx.conf 打开配置文件

- systemctl start nginx.service  启动nginx 服务
- cat /var/run/nginx.pid  查看nginx进程id


## 配置静态服务器
- cd /etc/nginx/conf.d  打开本文件夹

-  //在另一个回话框中执行新建目录操作
  
    - mkdir -p /data/images  创建静态文件目录images 存放图
    - mkdir -p /data/html
    - mkdir -p /data/download


- vi default.conf  默认配置文件
   将以下代码写入：

   ```
   // 定义集群（是为了实现服务器的负载均衡）xgjiqun  ，里面是对应的服务器（或服务）
      upstream xgjiqun{
        server localhost:4000;
        server localhost:5000;
        server localhost:6000;
        }

   ```

   ```
    // 一下代码写在server{}  的对象当中
      location ~ .*\.(jpg|png|gif)$ {
           
            valid_referers none blocked 47.104.184.134;// 设置防盗链（none blocked 47.104.184.134 符合三种条件之一就可以返回，否则是403）
            if ($invalid_referer) {
                return 403;
            }

            gzip off;  // 是否开启压缩
            gzip_http_version 1.1; // 版本号
            gzip_comp_level 3; // 压缩等级
            gzip_types image/jpeg image/png image/gif;  // 压缩资源匹配类型
            root /data/images;  // 文件放置目录
        }

        location ~ .*\.(html|js|css)$ {
            add_header Access-Control-Allow-Origin *; 允许所有域名访问（处理跨域问题，可以直接写允许访问的域名）
            add_header Access-Control-Allow-Methods GET,POST,PUT,DELETE,OPTIONS; // 允许访问的方法
            gzip on;
            gzip_min_length 1k;
            gzip_http_version 1.1;
            gzip_comp_level 9;
            gzip_types  text/css application/javascript;
            root /data/html;
        }

        location ~ ^/download {
            gzip_static on;  // 预压缩，当采用与压缩的时候，在下载的时候先找.gz结尾的文件解决cup的压缩时间
            tcp_nopush on; 
            expires 1h;// 给资源设置一个过期时间，1小时，用于缓存
            root /data/download;
        }
        
       // 正向代理（客户端代理）
       location ~ ^/api {  //设置代理 如果路径访问的 /api 那么就访问http://localhost:3000的服务器地址（在浏览器中输入192.168.33.10/api ，以为在虚拟主机中192.168.33.10是本地ip）
            porxy_pass http://loaclhost:3000  
        }

        // 处理集群--反向代理（服务器代理）
        location ~ ^/jiqun {
           proxy_pass http://xgjiqun;  // 使用上方定义的集群
        }


    ```     
    - nginx -t 检查配置文件是否正确
    - nginx -s reload  重启nginx







## 代理
- 正向： 代理的是客户端
- 反向：代理的是服务器

##  在liunx环境中安装node，安装node版本管理工具nvm 具体步骤
- yum update -y nss curl libcurl 升级culr 到最近版本
- curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.1/install.sh | bash   安装node 版本管理器 nvm
- tar -xzvf v0.23.0.tar.gz 解压压缩包
- 进入解压后的目录 ./install.sh
- source ~/bashrc
- nvm install node  安装node


## node进程管理工具（自动重启，负载均衡，性能监控等工作）
- npm install -g pm2