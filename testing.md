## 测试OSSEC rules/decoders
当用户在编写rules和decoders的时候，遇到的最大问题就是应该如何对这些规则进行测试。以前为了进行测试，可能需要重启很多次OSSEC，或者在重新安装一个测试版本进行测试。但是从版本1.6开始，有了一个简单的小工具来进行测试，那就是ossec-logtest。
## 利用ossec-logtest进行测试
