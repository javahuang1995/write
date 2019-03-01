```shell

#转到源目录：
cd /etc/yum.repos.d/

#之前的备份下
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.backup

#按照自己的版本下载源，我是CentOS 7，使用命令：
网易163源：
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo
阿里云源：
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
#-O,  --output-document=FILE    write documents to FILE.

运行以下命令生成缓存：
yum clean all
yum makecache

#试一下。。
yum install docker 

```

