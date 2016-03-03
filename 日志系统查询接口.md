日志系统查询接口
================
## RESTful API简要介绍
日志收集系统，使用强大的RESTful API进行数据查询操作。

这个接口可以实现相当部分的查询、聚合、计数等操作。此帖子将会在各个日志接入系统时不断补充。目前前端的日志查询与简单统计功能已经可以参考使用。
风格：
> curl -X\<verb\> '\<PROTOCAL\>://\<HOST\>:\<PORT\>/\<PATH\>?\<QUERY_STRING\>'  -d  '\<BODY\>'

VERB http方法：GET, POST, PUT, HEAD, DELETE
PROTOCOL http或者https协议
HOST
PORT
PATH
QUERY_STRING
BODY 

接口解释-举例：

所有功能：
查询过滤功能{

	1. 过滤term
	2. 多个值过滤terms
	3. 包含和不包含过滤 exists、missing。这两个字段只针对已经查出来了一批数据，但想区分某个字段是否存在时使用。
	4. 范围过滤range {
		包括的范围操作符有：
			gt：	大于
			lt：	小于
			gte：	大于等于
			lte：	小于等于
	}
}
过滤terms、聚合aggs、


1.查询该索引下一共有多少条日志记录：
curl -XGET 'http://test.el.daba.cn/_count
例如：
curl -XGET 'http://test.el.daba.cn/_search?pretty' -d {
	"query":{
		"match_all":{}
	}
}
返回：
<pre>
{
     "took": 5,
     "timed_out": false,
    "_shards": {
        "total": 40,
        "successful": 40,
        "failed": 0
    },
    "hits": {
        "total": 293812,
        "max_score": 1,
        "hits": [
            {
                "_index": "logstash-2016.01.31",
                "_type": "nginx_a_access",
                "_id": "AVMdQVo4H3aatJoSJ2B3",
                "_score": 1,
                "_source": {
                    "@version": "1",
                    "@timestamp": "2016-01-31T16:00:14.000Z",
                    "type": "log_nginx_a",
                    "host": "iZ255qhql7uZ",
                    "request_time": "0.000",
                    "res_size": "1251",
                    "remote_addr": "14.212.61.210",
                    "ident": "-",
                    "auth": "-",
                    "time_local": "01/Feb/2016:00:00:14 +0800",
                    "method": "GET",
                    "req_url_params": "/a.gif?id=11&uin=NaN&from=http://m.daba.cn/jsp/line/newlines.jsp?c=133&sr=1059&sc=803&ver=1.5.0&env=0&st=1454255953547&startcity=%E4%BD%9B%E5%B1%B1&arrivecity=%E5%8D%97%E5%AE%81&startdate=2016-02-01&presellday=10&ext={}&msg=TypeError: null is not an object (evaluating 'document.getElementById(\"fenxiang_id\").innerText')&target=http://m.daba.cn/jsp/line/newlines.jsp?c=133&sr=1059&sc=803&ver=1.5.0&env=0&st=1454255953547&startcity=%E4%BD%9B%E5%B1%B1&arrivecity=%E5%8D%97%E5%AE%81&startdate=2016-02-01&presellday=10&rowNum=1&colNum=39&level=4&_t=1454256014067",
                    "http_version": "1.1",
                    "status": "200",
                    "body_bytes_sent": "43",
                    "http_user_agent": "\"Mozilla/5.0 (iPhone; CPU iPhone OS 9_2_1 like Mac OS X) AppleWebKit/537.51.1 (KHTML, like Gecko) Version/4.0 Mobile/7A341 Safari/528.16\"",
                    "req_params": {
                        "id": "11",
                        "uin": "NaN",
                        "from": "http://m.daba.cn/jsp/line/newlines.jsp?c=133&sr=1059&sc=803&ver=1.5.0&env=0&st=1454255953547&startcity=%E4%BD%9B%E5%B1%B1&arrivecity=%E5%8D%97%E5%AE%81&startdate=2016-02-01&presellday=10",
                        "ext": "{}",
                        "msg": "TypeError: null is not an object (evaluating 'document.getElementById(\"fenxiang_id\").innerText')",
                        "target": "http://m.daba.cn/jsp/line/newlines.jsp?c=133&sr=1059&sc=803&ver=1.5.0&env=0&st=1454255953547&startcity=%E4%BD%9B%E5%B1%B1&arrivecity=%E5%8D%97%E5%AE%81&startdate=2016-02-01&presellday=10",
                        "rowNum": "1",
                        "colNum": "39",
                        "level": "4",
                        "_t": "1454256014067"
                    }
                }
            }
        ]
    }
}
</pre>
【注】：以上结果默认是10个，太占用篇幅，我给删掉了9个。当你希望返回任意多个时，在请求行参数列表中增加参数size=num即可。
样例2：
<pre>
curl -XGET 'http://test.el.daba.cn/_search?size=10&pretty' -d {
	"query":{
		"match_all":{}
	}
}
</pre>
###每天的PV
http://test.el.daba.cn/logstash-2016.02.25/_search?search_type=count //索引日期需要更改
{
	"aggs": {
		"PV": {
		    "terms": {
		        "field": "req_url_params.raw",		//字段值需要更改
		        "size": 4
		    } 
		}
	}
}
###每天的UV

###每天的IP

###前端错误msg+target:
1. 每天：
<pre>
http://test.el.daba.cn/logstash-2016.02.12/_search?search_type=count

{
	"aggs": {
		"msg": {
		    "terms": {
		        "field": "req_params.msg.raw",
		        "size": 2
		    } ,
    		"aggs": {
    		    "target":{
    		        "terms":{
    		            "field":"req_params.target.raw",
    		        "size": 2
    		        }
    		    }
    		}
		}
	}
}
</pre>

##按时间范围查询查询
{
    "aggs": {
        "yesterday": {
            "range": {
                "field": "@timestamp",
                "ranges": [
                    { "from": "2016-01-03T00:00:00+08:00", "to": "2016-01-04T23:59:59+0800" }
                ]
            }
        }
    }
}
##聚合的同时范围过滤
