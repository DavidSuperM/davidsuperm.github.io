# 1.查询1.12
http://altb1-comm-elasticsearch-1.vm.elenet.me:9200/ (es地址，后面的表述省略这一部分)
## 1.1查询全部
1. /your_index/_search    POST                      //查询该索引下的全部数据
  /your_index/your_type/_search    POST       //查询该索引下的type的全部数据
```
{}
```
2. /your_index/_search    POST 
 /your_index/your_type/_search    POST
```
{"query":{
      "match_all":{}
}}
```
## 1.2匹配查询（结果按得分排序）
 /your_index/_search    POST 
```
//match是先将关键词分段，然后分别匹配，查询，按得分排序
{
  "query": {
        "match" : {
            "id" : "1566613236964"
        }
    }
}
```

## 1.3半精确查询（查询包含value的文档）
 /your_index/_search    POST 
```
{
  "query": {
        "term" : {
            "label" : "标签1"
        }
    }
}
//如果label是not_analyzed（不分词），则这个就是精确查询，或者label的值都是纯数字，虽然分词器（默认是standard）会运行，但是对纯数字也不会分词，所以也是精确查询
//standard analyzer英文、数字按照空格来分词，中文直接使用一元分词
```

## 1.4嵌套半精确查询（查询包含value的文档）
/your_index/_search    POST
```
{
"query":{
 "nested":{
  "path":"paramContext.orders",
  "query":{
    "term":{
      "paramContext.orders.trackingId":
"3000105475638803345"
    }
   }
  }
 }
}
```

## 1.5匹配查询带返回个数
 /your_index/_search    POST 
```
{
  "query": {
        "match" : {
            "id" : "1566613236964"
        }
    }，
  "size": 2,
  "from": 0
}
```

## 1.6多字段查询
/your_index/_search    POST 
```
{
    "query": {
        "multi_match" : {
            "query" : "elasticsearch guide",
            "fields": ["title", "summary"]
        }
    }
}
```
## 1.7字符串的前缀查询
```
{"query":{
  "prefix":{
    "aoiId":"3430"
   }
}}
```

## 1.8模糊查询（通配符查询）
>(应该需要字段为String（其他字段是否可行还未一一验证），且进行了索引，index：no就是没有进行索引，无法查询)
```
{"query":{
  "wildcard":{
    "aoiId":"*43*"
   }
}}
```
## 1.9嵌套模糊查询
/your_index/_search    POST
```
{
"query":{
 "nested":{
  "path":"paramContext.orders",
  "query":{
    "wildcard":{
      "paramContext.orders.trackingId":
        "*"
    }
   }
  }
 }
}
```

## 1.10查询index下所有type的结构
/index_name/_mapping   GET
```
{}
```

## 1.11查看index的分词结果
/index_name/_analyze   POST
```
{
"field":"stationName",
"text":"贵溪支队"
}
```

## 1.12 查询数量
/your_index/_count    POST 
```
//match是先将关键词分段，然后分别匹配，查询，按得分排序
{
    "query": {
        "match" : {
        "id" : "1566613236964"
        }
    }
}
```

## 1.13 不用es head插件，直接http请求的方式查询
http://esip:端口/索引/_search?q=id:1566613236964

## 1.14 不用es head插件，http方式删除数据
```
curl -X POST "http://ip:port/索引/_delete_by_query" -H 'Content-Type: application/json' -d'
{
  "query":{
    "match":{
      "id":"101"
    }
  }
}
'
```


# 2.插入数据（没有该type时会自动创建type）
/index_name/type_name/   POST
```
{
"end_time":"2019-12-03T00:06:59",
"feature_group":"single_ets_user_address_feature",
"idc":"zb1",
"source_name":"zootopia-ekv",
"sql":"test1.sql",
"start_time":"2019-12-03T00:08:13",
"total":"365"
}
```
# 3.修改数据 
/index_name/type_name/_id/_update   POST
```
{
"end_time":"2019-12-03T00:06:59",
"feature_group":"single_ets_user_address_feature",
"idc":"zb1",
"source_name":"zootopia-ekv",
"sql":"test1.sql",
"start_time":"2019-12-03T00:08:13",
"total":"365"
}
```

# 4.删除数据
/index_name/type_name/_id/  DELETE
```
{}
```
# 5.更改结构

## 5.1新建索引
/index_name_new   PUT
```
{
	"mappings": {
		"type_name": {
			"properties": {
				"end_time": {
					"type": "date"
				},
				"source_name": {
					"type": "string"
				},
				"start_time": {
					"type": "date"
				}
			}
		}
	}
}
```

## 5.2新建type
/index_name/_mapping/type_name   PUT
```
{
  "properties" : {
    "tag" : {
      "type" :    "string"
    }
  }
}
```
## 5.3添加字段
/your_index/_mapping/your_type   PUT
```
{
     "properties": {
        "label": {
            "type": "string",
            "index":"not_analyzed" //不分词，这样搜索时该字段的value会作为一个整体检索，也就可以做到精确查询
        }
    }
}
//  好像设置type为keyword 也是不分词的意思
```
## 5.4删除、修改 字段、type
不支持，需要借助数据迁移来实现。
实现过程：需要先新建index，设置好正确的type，迁移原index所有数据，将原index删除，再将新index设置别名为原index的名字
工具:reindex

