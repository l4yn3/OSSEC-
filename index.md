## 安装所需环境
如果您使用的是UNIX相关的系统，那么只需要安装gnu make, gcc, and libc就够了。OpenSSL作为一个可选组件，我也建议您安装上。
## Ubuntu
在Ubuntu安装OSSEC，您只需要在系统上安装好*build-essential*即可。

执行以下命令安装*build-essential*。

> \#apt-get install build-essential

如果您的OSSEC需要数据库的支持（将检测信息存入数据库）,请先安装*mysql-dev* or *postgresql-dev*。使用如下命令安装*mysql-dev* or *postgresql-dev*。

> \# apt-get install mysql-dev postgresql-dev

## Redhat
Redhat系统默认情况下应该已经包含了所需要的安装组件。如果您需要数据库的支持，需要先安装*mysql-dev* or *postgresql-dev*。使用如下命令安装*mysql-dev* or *postgresql-dev*。
> \# yum install mysql-devel postgresql-devel

## Debian
Debian系统的终端系统默认用dash代替了hash。但是由于dash并不能完全支持其他shell终端所提供的所有功能，所以当安装agent的时候，设置服务器ip的时候可能会报一些错误信息。您可以忽略这些信息，但是您需要手动去设置一下服务器的ip地址。

打开agent的ossec.conf配置文件，确认是否存在以下信息，若不存在，在ossec.conf配置文件中添加以下信息。
> \<ossec_config> 
> 
>  \<client>
>
>   <server-ip>SERVER'S IP</server-ip>
>   
>  \</client>

或者直接在安装包内执行以下命令来避免以上比较麻烦的操作，用bash来直接安装agent。
> \# bash ./install.sh