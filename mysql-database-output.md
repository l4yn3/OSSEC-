## Mysql数据库支持
### 数据库设置
创建一个数据库，设置对应的用户，并导入表结构（表结构为源文件目录src/os_dbd下的mysql.schema）。
	
	# mysql -u root -p
	
	mysql> create database ossec;
	
	mysql> grant INSERT,SELECT,UPDATE,CREATE,DELETE,EXECUTE on ossec.* to ossecuser@<ossec ip>;
	Query OK, 0 rows affected (0.00 sec)
	
	mysql> set password for ossecuser@<ossec ip>=PASSWORD('ossecpass');
	Query OK, 0 rows affected (0.00 sec)
	
	mysql> flush privileges;
	Query OK, 0 rows affected (0.00 sec)
	
	mysql> quit
	
	# mysql -u root -p ossec < mysql.schema

## 设置OSSEC
为了将报警信息和其他数据存入到数据库当中，我们需要在OSSEC的配置文件（/var/ossec/etc/ossec.conf)添加\<database_output\>节点。

	<ossec_config>
	    <database_output>
	        <hostname>192.168.2.30</hostname>
	        <username>ossecuser</username>
	        <password>ossecpass</password>
	        <database>ossec</database>
	        <type>mysql</type>
	    </database_output>
	</ossec_config>

hostname为数据库主机名称，username为mysql用户，password为对应的密码，database为对应的数据库。

## 完成Mysql存储
接着我们执行以下命令，完成Mysql存储。

	# /var/ossec/bin/ossec-control enable database
	# /var/ossec/bin/ossec-control restart