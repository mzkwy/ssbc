# ssbc
手撕包菜磁力链接搜索网站

## 网站说明
这是 www.shousibaocai.com 的网站源代码。
开源的目的是为了促进技术交流和相互学习，把DHT与搜索引擎技术应用到更广泛的领域去。

##一键安装脚本
```bash

#!/bin/bash
#changelog:
#1.1添加开机自启动功能
#1.2修改pip获取方式
#1.3添加linode主机repo源的问题，改为用阿里云的repo源
python -V          
systemctl stop firewalld.service  
systemctl disable firewalld.service   
systemctl stop iptables.service  
systemctl disable iptables.service  
yum -y install wget
#如果使用linode主机，请取消下面4行的注释
# wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# wget -qO /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
# yum clean metadata
# yum makecache
cd ~
wget https://github.com/78/ssbc/archive/master.zip
yum -y install unzip
unzip master.zip
#解压后 源码在/root/ssbc-master目录
yum -y install gcc
yum -y install gcc-c++
yum -y install python-devel
yum -y install mariadb
yum -y install mariadb-devel
yum -y install mariadb-server
cd ssbc-master
yum -y install epel-release
yum -y install  python-pip
pip install -r requirements.txt
pip install  pygeoip
systemctl start  mariadb.service 
mysql -uroot  -e"create database ssbc default character set utf8;"  
sed -i '/!includedir/a\wait_timeout=288000\ninteractive_timeout = 288000\nmax_allowed_packet = 256M' /etc/my.cnf
mkdir  -p  /data/bt/index/db /data/bt/index/binlog  /tem/downloads
chmod  755 -R /data
chmod  755 -R /tem
yum -y install unixODBC unixODBC-devel postgresql-libs
wget http://sphinxsearch.com/files/sphinx-2.2.11-1.rhel7.x86_64.rpm
rpm -ivh sphinx-2.2.9-1.rhel7.x86_64.rpm
systemctl restart mariadb.service  
systemctl enable mariadb.service 
searchd --config ./sphinx.conf
python manage.py makemigrations
python manage.py migrate
indexer -c sphinx.conf --all 
ps aux|grep searchd|awk '{print $2}'|xargs kill -9
searchd --config ./sphinx.conf
#supervisor
yum -y install supervisor
supervisord -c /etc/supervisord.conf
yum -y install lrzsz

#创建账户密码
cd ..
python manage.py createsuperuser
#开机自启动
chmod +x /etc/rc.d/rc.local
echo "systemctl start  mariadb.service " >> /etc/rc.d/rc.local
echo "cd /root/ssbc-master " >> /etc/rc.d/rc.local
echo "indexer -c sphinx.conf --all " >> /etc/rc.d/rc.local
echo "searchd --config ./sphinx.conf " >> /etc/rc.d/rc.local
echo "supervisorctl update " >> /etc/rc.d/rc.local

```
