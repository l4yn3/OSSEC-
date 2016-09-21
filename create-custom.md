## 创建decoder和rules
监控系统和程序日志是OSSEC的主要功能之一。非常多的通用程序都有logs和decoders。为没有logs和decoders的应用和服务添加decoders和rules也是非常简单的。
##添加被监控文件
添加一个新的日志文件到监控系统是非常简单的。在OSSEC系统的ossec.conf文件中添加如下代码。
> \<localfile>
> 
>  \<log_format>syslog\</log_format>
>  
>  \<location>/path/to/log/file\</location>
>  
>\</localfile>

syslog是一种通用日志格式（添加指定格式文本到日志文本文件当中）。

添加完如上代码之后，就把/path/to/log/file添加到了监控系统当中。重启OSSEC程序才能生效。

##创建一个标准Decoder
接下来的文档中，我们都将用如下的日志信息作为演示内容。

    2013-11-01T10:01:04.600374-04:00 arrakis ossec-exampled[9123]: test connection from 192.168.1.1 via test-protocol1

    2013-11-01T10:01:05.600494-04:00 arrakis ossec-exampled[9123]: successful authentication for user test-user from 192.168.1.1 via test-protocol1

第一行日志逐个字段分析如下：


-  2013-11-01T10:01:04.600374-04:00 - 日志生成时间戳
-  arrakis - 系统名称
-  ossec-exampled - 生成此日志的守护进程
-  [9123] - ossec-exampled的具体进程id
-  test connection from 192.168.1.1 via test-protocol1 - 日志内容

我们用 *ossec-logtest* 来测试新编写的decoder和rules。
默认包含的decoder都在local_decoder.xml文件当中（如果您选择默认安装，位置应该是/var/ossec/etc）。

我们用ossec-logtest来测试我们以上这两条日志信息，采用的decoder就是默认的decoder。

    # /var/ossec/bin/ossec-logtest
     
    2013/11/01 10:39:07 ossec-testrule: INFO: Reading local decoder file.

    2013/11/01 10:39:07 ossec-testrule: INFO: Started (pid: 32109).
    ossec-testrule: Type one log per line.

    2013-11-01T10:01:04.600374-04:00 arrakis ossec-exampled[9123]: test connection from 192.168.1.1 via test-protocol1


     **Phase 1: Completed pre-decoding.

     full event: '2013-11-01T10:01:04.600374-04:00 arrakis ossec-exampled[9123]: test connection from 192.168.1.1 via test-protocol1'
     hostname: 'arrakis'
     program_name: 'ossec-exampled'
     log: 'test connection from 192.168.1.1 via test-protocol1'

     **Phase 2: Completed decoding.
     No decoder matched.

屏幕上输出的内容很少，这是因为OSSEC系统默认的规则并不能很好的理解我们这两条日志。为了让OSSEC了解关于这两条日志的更多信息，我们必须自己编写新的decoder。

我们为ossec-exampled程序添加一个很简单的decoder。在decoder.xml文件当中添加如下代码。

	<decoder name="ossec-exampled">
	  <program_name>ossec-exampled</program_name>
	</decoder>

这个decoder看起来就是为了获取所有ossec-exampled生成的日志。启用这样一个简单的decoder，可以让OSSEC用户在之后编写出更详细匹配日志的decoder。

下面内容是添加了上边decoder之后的标准输出。

		# /var/ossec/bin/ossec-logtest
		2013/11/01 10:52:09 ossec-testrule: INFO: Reading local decoder file.
		2013/11/01 10:52:09 ossec-testrule: INFO: Started (pid: 25151).
		ossec-testrule: Type one log per line.
		
		2013-11-01T10:01:04.600374-04:00 arrakis ossec-exampled[9123]: test connection from 192.168.1.1 via test-protocol1
		
		
		**Phase 1: Completed pre-decoding.
        full event: '2013-11-01T10:01:04.600374-04:00 arrakis ossec- exampled[9123]: test connection from 192.168.1.1 via test-  protocol1'
        hostname: 'arrakis'
        program_name: 'ossec-exampled'
        log: 'test connection from 192.168.1.1 via test-protocol1'

		**Phase 2: Completed decoding.
        decoder: 'ossec-exampled'

Phase 2 准确的识别出了这条日志来自于ossec-exampled应用程序。这样还远远不够，这条日志还有很多信息可以被我们识别出来，比如test-protocol1 和 source IP 。因此我们需要创建一个刚才decoder的子decoder，用*prematch*标签匹配出更详细的日志信息。

	<decoder name="ossec-exampled-test-connection">
	  <parent>ossec-exampled</parent>
	  <prematch offset="after_parent">^test connection </prematch> <!-- offset="after_parent" makes OSSEC ignore anything matched by the parent decoder and before -->
	  <regex offset="after_prematch">^from (\S+) via (\S+)$</regex> <!-- offset="after_prematch" makes OSSEC ignore anything matched by the prematch and earlier-->
	  <order>srcip, protocol</order>
	</decoder>

让我们详细解释一下以上decoder。

- \<decoder name="ossec-exampled-test-connection">-表示这是一个decoder，并且将这个decoder命名为 ossec-exampled-test-connection。
- \<parent>ossec-exampled\</parent> - 将ossec-exampled作为父decoder，即表示ossec-exampled被触发以后，才能触发这个decoder。
- \<prematch offset="after_parent">^test connection \</prematch> - 如果此标签内的内容得不到匹配，那么将不会启用这个decoder。offset标签是表示当父decoder匹配完之后再进行这项数据对比检查（目的是加快内容匹配）。
- \<regex offset="after_prematch">^from (\S+) via (\S+)$\</regex> - 这行正则表达式用来获取匹配数据，并将数据应用在rules当中。在这个例子当中，\S+匹配的是IP地址，第二处匹配的是相关协议。任何括号当中的内容都可以被用于rules的规则当中。
- \<order>srcip, protocol\</order> - 这个标签内的内容，用来定义上一个regex标签内正则所匹配括号内的内容分别代表什么。比如这个例子，regex当中括号内的正则分别代表了IP(srcip)和协议（protocal）。

添加完以上decoder，我们用ossec-logtest来测试一下。

	# /var/ossec/bin/ossec-logtest
	2013/11/01 11:03:25 ossec-testrule: INFO: Reading local decoder file.
	2013/11/01 11:03:25 ossec-testrule: INFO: Started (pid: 6290).
	ossec-testrule: Type one log per line.
	
	2013-11-01T10:01:04.600374-04:00 arrakis ossec-exampled[9123]: test connection from 192.168.1.1 via test-protocol1
	
	
	**Phase 1: Completed pre-decoding.
	       full event: '2013-11-01T10:01:04.600374-04:00 arrakis ossec-exampled[9123]: test connection from 192.168.1.1 via test-protocol1'
	       hostname: 'arrakis'
	       program_name: 'ossec-exampled'
	       log: 'test connection from 192.168.1.1 via test-protocol1'
	
	**Phase 2: Completed decoding.
	       decoder: 'ossec-exampled'
	       srcip: '192.168.1.1'
	       proto: 'test-protocol1'



Note

	在Phase 2的输出当中，decoder被标示成了ossec-exampled,有人可能会很疑惑是不是子decoder没有执行。这里其实子decoder是被执行了的。


刚才我们用ossec-logtest测试了第一条log信息，并且成功了。那么我们再用ossec-logtest来测试一下第二条log信息。
2013-11-01T10:01:05.600494-04:00 arrakis ossec-exampled[9123]: successful authentication for user test-user from 192.168.1.1 via test-protocol1


	**Phase 1: Completed pre-decoding.
	       full event: '2013-11-01T10:01:05.600494-04:00 arrakis ossec-exampled[9123]: successful authentication for user test-user from 192.168.1.1 via test-protocol1'
	       hostname: 'arrakis'
	       program_name: 'ossec-exampled'
	       log: 'successful authentication for user test-user from 192.168.1.1 via test-protocol1'
	
	**Phase 2: Completed decoding.
	       decoder: 'ossec-exampled'

我们在decoder  ossec-exampled-test-connection当中添加的规则，并没有像上一条一样被解析出来。这是结果是在我们预料之中的，因为 prematch 标签并没有从这条日志当中匹配到内容。实际上这条日志当中，有4个地方对我们来说是有用的信息：status (successful), srcuser, srcip, and protocol。那就让我们再次添加一个decoder来匹配这条日志。

	<decoder name="ossec-exampled-auth">
	  <parent>ossec-exampled</parent>
	  <prematch offset="after_parent"> authentication </prematch>
	  <regex offset="after_parent">^(\S+) authentication for user (\S+) from (\S+) via (\S+)$</regex> <!-- Using after_parent here because after_prematch would eliminate the possibility of matching the status (successful) -->
	  <order>status, srcuser, srcip, protocol</order>
	</decoder>

ossec-logtest的输出为：

	2013-11-01T10:01:05.600494-04:00 arrakis ossec-exampled[9123]: successful authentication for user test-user from 192.168.1.1 via test-protocol1


	**Phase 1: Completed pre-decoding.
	       full event: '2013-11-01T10:01:05.600494-04:00 arrakis ossec-exampled[9123]: successful authentication for user test-user from 192.168.1.1 via test-protocol1'
	       hostname: 'arrakis'
	       program_name: 'ossec-exampled'
	       log: 'successful authentication for user test-user from 192.168.1.1 via test-protocol1'
	
	**Phase 2: Completed decoding.
	       decoder: 'ossec-exampled'
	       status: 'successful'
	       srcuser: 'test-user'
	       srcip: '192.168.1.1'
	       proto: 'test-protocol1'


Phase 2当中我们成功的提取出了我们想要的这些数据信息。我们再次用此规则测试一下之前的log信息，以确保之前的测试不会出现什么问题。

	2013-11-01T10:01:04.600374-04:00 arrakis ossec-exampled[9123]: test connection from 192.168.1.1 via test-protocol1
	
	
	**Phase 1: Completed pre-decoding.
	       full event: '2013-11-01T10:01:04.600374-04:00 arrakis ossec-exampled[9123]: test connection from 192.168.1.1 via test-protocol1'
	       hostname: 'arrakis'
	       program_name: 'ossec-exampled'
	       log: 'test connection from 192.168.1.1 via test-protocol1'
	
	**Phase 2: Completed decoding.
	       decoder: 'ossec-exampled'
	       srcip: '192.168.1.1'
	       proto: 'test-protocol1'