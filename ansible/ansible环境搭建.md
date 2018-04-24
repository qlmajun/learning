### ansible环境搭建 ###
-----

### 1、设置EPEL仓库
```
wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```

### 2、安装EPEL
```
 rpm -ivh epel-release-latest-7.noarch.rpm
```

### 3、安装ansible
```
yum -y install ansible
```

### 4、ansible版本查看
```
ansible --version
```
