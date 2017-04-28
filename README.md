# sphinx-chinese
给官方的 sphinx 增加对中文的支持（目前只支持utf8），基于流行的中文分词插件 https://github.com/hightman/scws

# 安装方法
## 安装scws
可以参考 https://github.com/hightman/scws 里的安装方法
```bash
$ cd /tmp
$ wget http://www.xunsearch.com/scws/down/scws-1.2.1.tar.bz2 
$ tar xjf scws-1.2.1.tar.bz2 
$ cd scws-1.2.1
$ #默认路径(/user/local)就简单的 ./configure --enable-static
$ ./configure --prefix=/tmp/scws --enable-static #需要静态编译
$ make 
$ make install
$ tree /tmp/scws
```
## 安装sphinx
源码存储在 https://github.com/hetao29/sphinx ，这是官方的分支
```bash
$ cd /tmp
$ git clone https://github.com/hetao29/sphinx.git sphinx-chinese
$ cd sphinx-chinese
$ #请使用-chinese的分支，目前有2个，master-chinese, rel22-chinese
$ git checkout rel22-chinese 
$ #如果默认安装就是 --with-scws=/usr/local
$ ./configure --prefix=/tmp/sphinx_bin/ --with-scws=/tmp/scws/ 
$ make 
$ make install
```

## 测试
```bash
$ cd /tmp
$ git clone https://github.com/hetao29/sphinx-chinese.git sphinx-test
$ cd sphinx-test/test
$ make  #测试
```

## 配置sphinx
具体的手册说明参考 http://www.xunsearch.com/scws/docs.php 
```bash
$ cat /tmp/sphinx-test/test/course.conf
```
```conf
source course
{
	type			= xmlpipe2
	xmlpipe_command		=  cat /tmp/course.xml
	xmlpipe_fixup_utf8	= 1
}
index course
{
	type			= plain
	source			= course
	path			= /tmp/test_chinese
	scws = 1
	scws_dict=/tmp/dict.utf8.txt
	scws_rule=/tmp/rules.utf8.ini
	scws_multi=11
}


indexer
{
	mem_limit		= 512M
}

searchd
{
	listen			= 9312
	log			= /tmp/searchd.log
	query_log		= /tmp/query.log
	read_timeout		= 5
	client_timeout		= 300
	max_children		= 30
	pid_file		= /tmp/searchd.pid
	#max_matches		= 10000
	seamless_rotate		= 1
	preopen_indexes		= 1
	unlink_old		= 1
	mva_updates_pool	= 1M
	max_packet_size		= 8M
	max_filters		= 256
	max_filter_values	= 4096
	max_batch_queries	= 32
	workers			= threads # for RT to work
}
```
```sh
# 参数说明
#支持如下4个参数

#总开关，必须设置，注释掉就关掉scws支持，任意值都是开启
scws = 1 

#词库设置，可选，如果不设置，默认全是单字切分，请使用utf8格式的词库
#可以是xdb格式或者文本格式，根据扩展名(.xdb)自动识别
#如果不是xdb格式的文件，第一次使用时会在/tmp目录自动生成xdb格式并优化，会有一点耗时，后面就不会了
#词典格式类型说明
#
# SCWS_XDICT_XDB      1
# SCWS_XDICT_MEM      2
# SCWS_XDICT_TXT      4

#可以使用dict目录下的dict.utf8.txt
scws_dict=/tmp/dict.utf8.txt

#规则文件，可选
#可以使用dict目录下的 rules.utf8.in
scws_rule=/tmp/rules.utf8.ini


#复合切词，可选，默认为0，缺省不复合分词。取值由下面几个常量异或组合（也可用 1-15 来表示，就是数字相加，比如3就表示1+2）：
#推荐3(主要是词组为主)或者11(包含了全部单字)
#
# SCWS_MULTI_SHORT   (1)短词
# SCWS_MULTI_DUALITY (2)二元（将相邻的2个单字组合成一个词）
# SCWS_MULTI_ZMAIN   (4)重要单字
# SCWS_MULTI_ZALL    (8)全部单字
scws_multi=3
    
```
# 词库说明
#生成词库和词库格式说明
参考 https://github.com/hetao29/sphinx-chinese/tree/master/tools
