模仿[logstash](https://www.elastic.co/products/logstash)做的一个应用.

我们一直用logstash从Kafka消费数据进ES, 随着数据量的越来越大, 需要的logstash实例和机器也越来越多. 

于是就拿java实现了一下目前我们用到的Logstash的插件.   

- input
	 * kafka
- output
	* elastickseach
	* kafka
	* stdout  
- filter
	* Grok
	* Date 
	* Json
	* Gsub
	* Drop
	* Trim
	* Translate
	* Rename
	* 等

用一个典型配置做测试, 包括Grok Date AddField If条件判断等, 吞吐量是Logstash的5倍左右 .  
不压测吞吐量, 正常消费的情况下, CPU使用率大概是Logstash的50%到25%.

# 配置
配置在一定程度上也是模仿logstash, 但用的是通用的yaml格式.
因为能力有限, 不能实现logstash的那套语法, if是用了另外的格式.
[可以参考一个配置的示例](https://github.com/childe/hangout/blob/master/example.yml)

## Input
### kafka

	topic: test # string , no default
	groupID: hangout # string, no default
	zk: 192.1.1.100:2181 #string, no default
	threads: 1 # default 1
	session_timeout: "6000" # string, default 6000
	commit_interval: "60000" # string, auto.commit.interval.ms, default 1 minute.
	codec: plain # string, default json


## Filter
### Grok
Grok是为了把一个字符串切割成多个field, 用的是[Joni库](https://github.com/jruby/joni), 暂不支持logstash里面的patterns.

会依次匹配match中的正则, 直到有一个成功的.

**注意, 如果正则中的groupname和已有的字段一样, 原来的字段被覆盖**

	src: message #default message
	match:
		- '(?<logtime>\S+) (?<user>\S+) (-|(?<level>\w+))'
		- '(?<logtime>\S+\s+\S+) (?<user>\S+) (-|(?<level>\w+))'
	remove_fields: ['message']
	tagOnFailure: grokfail #default grokfail

### Date

### freemarker
If 条件和addfileds等插件使用[freemarker](http://freemarker.org/).
[在线测试](http://freemarker-online.kenshoo.com/)