#### 0、常用命令

~~~shell
# 常用指令
rm -rf /var/run/yum.pid             # 关闭yum进程
echo '123456'|passwd --stdin root   #设置密码
# 关闭irewalld防火墙
systemctl stop firewalld.service    # 停止firewalld服务
systemctl disable firewalld.service # 开机禁用firewalld服务

vi /etc/sysconfig/selinux           # 编辑selinux文件 Esc按:wq保存并退出 i进入编辑模式
SELINUX=disabled                    # 把文件中的SELINUX=enforcing 改成 SELINUX=disabled 即可
sestatus                            # 查看SELinux状态
~~~

#### 1、安装java

~~~shell
# 切换root
sudo su
# 查看目前安装的java
rpm -qa | grep java
# 卸载自带java
yum -y remove java-1.8.0-openjdk*
yum -y remove tzdata-java.noarch
# 查看yum库可安装版本
yum -y list java*
# 安装java1.8
yum install java-1.8.0-openjdk* -y
# 安装完成使用javac和java -version测试
javac
java -version
~~~

#### 2、安装redis

~~~shell
# redis官网找到下载链接 redis.io 或者 redis.cn
# 新建redis文件夹并打开到终端，下载
wget https://download.redis.io/releases/redis-6.2.1.tar.gz
#redis
tar -zxvf redis-6.2.1.tar.gz
#编译redis
cd redis-6.2.1
make
#修改配置文件开启失效key回调并设置后台启动
cd redis-6.2.1
vim redis.conf（daemonize yes，notify-keyspace-events Ex）
#启动redis（可忽略）
/home/benben/redis/redis-6.2.1/src/redis-server /home/benben/redis/redis-6.2.1/redis.conf
#设置自启动
cd utils
./install_server.sh
#按照提示确定或输入对应地址
redis executable path 为 /usr/local/bin/redis-server 
配置文件为/home/benben/redis/reids-6.2.1/redis.conf
#查看运行状态
systemctl status redis_6379
~~~

```shell
# 如果遇到：This systems seems to use systemd.Please take a look at the provided example service unit files in this directory, and adapt and install them. Sorry!
# 则需注释install_server.sh中的以下几行代码：

    #bail if this system is managed by systemd
    #_pid_1_exe="$(readlink -f /proc/1/exe)"
    #if [ "${_pid_1_exe##*/}" = systemd ]
    #then
    #       echo "This systems seems to use systemd."
    #       echo "Please take a look at the provided example service unit files in this directory, and adapt and install them. Sorry!"
    #       exit 1
    #fi
```
#### 3、安装MySQL

~~~shell
#MySQL官网查看下载链接（https://dev.mysql.com/downloads/repo/yum/）
# 下载
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
rpm -ivh mysql80-community-release-el7-3.noarch.rpm
# 安装服务
cd  /etc/yum.repos.d/
yum -y install mysql-server
# 启动并获取临时密码登录
systemctl start mysqld
grep 'temporary password' /var/log/mysqld.log
mysql -uroot -p（密码VN(l->W=:3oi）
# 修改密码
set global validate_password_policy=LOW;（密码低风险）（高版本为set global validate_password.policy=LOW;）
set global validate_password_length=6;（密码长度）（高版本为set global validate_password.length=6;）
ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
# 设置远程访问
use mysql;（切换MySQL数据）
select Host,User from user;（查看user表）
update user set Host='%' where User='root';（修改为允许任何地址访问）
flush privileges;（刷新权限）
~~~

#### 4、安装Minio

~~~shell
# 下载并启动
wget https://dl.min.io/server/minio/release/linux-amd64/minio
chmod +x minio
/home/benben/minio/minio server /home/benben/minio/data
# 后台启动
nohup /home/benben/minio//minio server /home/benben/minio/data > /home/benben/minio/data/minio.log 2>&1 &
# 开机自启动
export MINIO_ACCESS_KEY=minioadmin
export MINIO_SECRET_KEY=minioadmin
nohup /home/benben/minio//minio server  /home/benben/minio/data > /home/benben/minio/data/minio.log 2>&1 &
# 加入以上命令后赋权并重启
vi /etc/rc.local
chmod +x /etc/rc.local
~~~

#### 5、安装RabbitMQ 

~~~shell
# 下载和安装erlong（https://github.com/rabbitmq/erlang-rpm）
wget https://github.com/rabbitmq/erlang-rpm/releases/download/v23.3.1/erlang-23.3.1-1.el7.x86_64.rpm
rpm -ivh erlang-23.3.1-1.el7.x86_64.rpm
# 下载和编译安装socat（http://repo.iotti.biz/CentOS/7/x86_64/）
wget http://repo.iotti.biz/CentOS/7/x86_64/socat-1.7.3.2-5.el7.lux.x86_64.rpm
rpm -ivh socat-1.7.3.2-5.el7.lux.x86_64.rpm
# 下载安装rabbitmq（https://github.com/rabbitmq/rabbitmq-server）
wget https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.14/rabbitmq-server-3.8.14-1.el7.noarch.rpm
rpm -ivh rabbitmq-server-3.8.14-1.el7.noarch.rpm
# 启动管理页面和服务和stomp
rabbitmq-plugins enable rabbitmq_management
rabbitmq-plugins enable rabbitmq_web_stomp
service rabbitmq-server start
# 开启远程访问
cd /etc/rabbitmq
touch rabbitmq.config
vim rabbitmq.config
写入[{rabbit, [{loopback_users, []}]}].
service rabbitmq-server restart
# 设置用户和密码
# 删除guest用户
rabbitmqctl delete_user guest
rabbitmqctl add_user admin 123456
rabbitmqctl set_user_tags admin administrator
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"
# 开机自启动
systemctl daemon-reload
systemctl enable rabbitmq-server
systemctl start rabbitmq-server
~~~

#### 6、安装ELK

~~~shell
# 下载和安装    https://www.elastic.co/cn/downloads/past-releases
# 导入秘钥
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
wget https://artifacts.elastic.co/downloads/kibana/kibana-7.12.0-x86_64.rpm
wget https://artifacts.elastic.co/downloads/logstash/logstash-7.12.0-x86_64.rpm
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.12.0-x86_64.rpm
rpm -ivh elasticsearch-7.12.0-x86_64.rpm
rpm -ivh kibana-7.12.0-x86_64.rpm
rpm -ivh logstash-7.12.0-x86_64.rpm
# 设置开机启动 
# 注意elasticsearch占用内存很大 可修改调整conf/jvm.options -Xms和-Xmx
systemctl daemon-reload
systemctl enable elasticsearch
systemctl start elasticsearch
systemctl daemon-reload
systemctl enable kibana
systemctl start kibana
systemctl daemon-reload
systemctl enable logstash
systemctl start logstash
sudo /usr/share/logstash/bin/system-install /etc/logstash/startup.options systemd
~~~

