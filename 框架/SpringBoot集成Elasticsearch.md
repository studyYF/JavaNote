- 分词器搜索全匹配查询
- 低版本springboot集成es问题
- logstash同步问题

> Elasticsearch是目前比较火的搜索引擎，能够做到快速的全文检索。本文不涉及ES的原理等基础知识，只是一篇关于SpringBoot如何集成Elasticsearch、使用logstash如何同步mysql数据库中的数据到Elasticsearch的简单入门教程。

####  版本匹配

`SpringBoot`提供了`spring-boot-starter-data-elasticsearch`对`Elasticsearch`的使用进行了封装，可以快速方便的使用提供的`API`进行操作，这是最简单的集成以及操作`Elasticsearch`方法。但是对于`SpringBoot`以及Elasticsearch的版本有要求，由于目前我们公司使用的还是`SpringBoot1.5.3`的版本，对应的`starter`只能支持`Elasticsearch5.0`以下的版本，所以不能使用最新的7.x的`Elasticsearch`。

| SpringBoot Version x | spring-boot-starter-data-elasticsearch Version y | Elasticsearch Version z |
| :------------------: | :----------------------------------------------: | :---------------------: |
|      x < 2.0.0       |                    y < 2.0.0                     |         z < 5.0         |
|      x >= 2.0.0      |                    y >= 2.0.0                    |         z > 5.0         |

升级项目中的`Springboot`版本不太现实，而又想使用最新的`Elasticsearch`，只能换一种方式集成，`restClient`的集成方式，这种方式对于版本的兼容性较好。`restClient`有两种，一种是low-level，一种是high-level，两者的原理基本一致，区别最大的是封装性，官方建议使用`high-level`，而且`low-level`将逐渐被废弃，所以我们使用`elasticsearch-rest-high-level-client`进行集成。

```xml	
<dependency>
  <groupId>org.elasticsearch.client</groupId>
  <artifactId>elasticsearch-rest-high-level-client</artifactId>
  <version>7.6.0</version>
</dependency>
```

**对于这种框架的整合，各种组件的版本一定要匹配上，如果不能对应，会出现各种意想不到的情况，在这里我也是走了很多弯路才搞清楚。**

#### Elasticsearch基本使用

##### 配置host和端口

`Elasticsearch`默认的端口是9200和9300，9200是提供给`http`方式连接的，9300对应的是`tcp`的方式连接，这里我们使用9200。

```yaml
spring:
	elasticsearch:
    host: 192.168.3.75
    port: 9200
```

##### 注入restHighLevelClient

新建一个配置类，读取``host``和`port`，并创建一个`restHighLevelClient`的`bean`注入到`spring`容器中。

```java
@Configuration
public class EsConfig {
    @Value("${spring.elasticsearch.port}")
    private String port;
    @Value("${spring.elasticsearch.host}")
    private String host;
    @Bean
    public RestHighLevelClient restHighLevelClient() {
        return new RestHighLevelClient(RestClient.builder(new HttpHost(host,Integer.parseInt(port))));
    }
}
```

##### 使用client进行查询

```java
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
searchSourceBuilder.query(QueryBuilders.matchQuery("name","123"));
SearchRequest searchRequest = new SearchRequest("test_index");
searchRequest.source(searchSourceBuilder);
try {
  SearchResponse searchResponse = restHighLevelClient.search(searchRequest, RequestOptions.DEFAULT);
  searchResponse.getHits().forEach(i -> {
    String jsonString = i.getSourceAsString();
    //使用fastjson将json对象转化为model
    EsGoodsModel esGoodsModel = JSONObject.parseObject(jsonString,EsGoodsModel.class);
  });
} catch (IOException e) {
  e.printStackTrace();
}
```

这里只是简单的展示了基本的使用，具体的查询条件的封装，像分页、排序、条件查询等都差不多。



#### 使用Logstash同步数据

##### 关于logstash

关于搜索数据的导入，我这里使用了官方推荐的`logstash`，还有一些其他的方式，这里不做赘述。

使用`logstash`最重要的是写好`conf`文件。他的格式如下

```json
input {
  
}
filter {
  
}
output {
  
}
```

- `input`表示输入的数据来源，可以是`file`、`jdbc`、`http`、`kafka`、`log4j`、`redis`等很多途径（具体可以查看

[官网](https://www.elastic.co/guide/en/logstash/current/input-plugins.html)）。

- `filter`主要是对数据来源进行过滤，转换成`json`格式，然后保存到`Elasticsearch`中。`filter`里面有很多的插件，具体官网有详细的介绍，本次教程主要使用到`Aggregate`聚合数据。

- `output`是数据输出到哪里，也有很多中，本次使用输出到`Elasticsearch`中。

##### 数据要求

目前我们项目中使用到的是对商品进行检索，商品中有一些属性是来自于其他表，且可能有多条数据，类似下面的数据结构。

```json
{
	"stock_info" : "300公斤",
  "name" : "黄瓜",
  "address" : "上海市普陀区",
  "price" : "1.59",
  "company_name" : "供应商",
  "number" : "SP484",
  "plant_area" : "50",
  "id" : "55010b5154f84a2fbec4056c185789ac",
  "sl_url" : "黄瓜1_1584424256774.jpg",
  "type" : 1,
  "goodsLabelList" : [
    {
      "dictionary_value" : "绿色"
    },
    {
      "dictionary_value" : "有机"
    }
  ],
  "attributeValueList" : [
    {
      "attribute_value" : "密刺黄瓜",
      "attribute_id" : "68a40212b85c41019f843f8934bbbda5"
    },
    {
      "attribute_value" : "严重皱缩",
      "attribute_id" : "d50368f5fab442808dd27ee2c5361048"
    },
    {
      "attribute_value" : "15~25cm",
      "attribute_id" : "58771cbc4caf41789cf747c13fe755bb"
    }
  ]
}
```

像这种`goodsLabelList`对应于`Elasticsearch`就是嵌套的数据类型。下面就需要配置logstash的conf文件。

`input`使用 `jdbc`插件进行输入

```json
input {
  jdbc {
  	#这里指定connector的位置
  	jdbc_driver_library => "../mysql-connector-java-5.1.43-bin.jar"
    jdbc_driver_class => "com.mysql.jdbc.Driver"
    jdbc_connection_string => "jdbc:mysql://192.168.3.78:3306/guoxn_bab_test?useSSL=false&serverTimezone=UTC&rewriteBatchedStatements=true&characterEncoding=utf8"
    jdbc_user => "**"
    jdbc_password => "**"
  	#定时时间，一分钟一次
    schedule => "* * * * *"
    jdbc_paging_enabled => "true"
    jdbc_page_size => "50000"
    record_last_run => true
    use_column_value => true
  	#设置时区，如果默认会有8个小时的时差
    jdbc_default_timezone => "Asia/Shanghai"
  	#这里保存last_update_time也可以不指定
    last_run_metadata_path => "../last_goods_record.txt"
  	#根据last_update_time进行更新数据
    tracking_column => "last_update_time"
    tracking_column_type => "timestamp"
  	#sql省去了具体的查询内容，主要注意 :sql_last_value 的写法
    statement => "
                SELECT .... and t1.last_update_time > :sql_last_value and t1.last_update_time < NOW() AND t1.is_delete=0 AND t1.type=1 order by t1.id desc"
	}
}
```

`jdbc`的输入基本没有什么问题，主要是下面的`filter`部分。

```json
filter {
  			#使用aggregate进行聚合数据
        aggregate {
  					#task_id指定任务的id，来自于上面jdbc的sql查询结果，进行聚合的时候，一定要按照id进行排序，不然可能导致数据的丢失
            task_id => "%{id}"
            code => "
                map['id'] = event.get('id')
                map['name'] = event.get('name')
                map['sl_url'] = event.get('sl_url')
                map['company_name'] = event.get('company_name')
                map['description'] = event.get('description')
                map['price'] = event.get('price')
                map['stock_info'] = event.get('stock_info')
                map['address'] = event.get('address')
                map['number'] = event.get('number')
                map['shelf_time'] = event.get('shelf_time')
                map['shelf_stock'] = event.get('shelf_stock')
                map['company_id'] = event.get('company_id')
                map['plant_area'] = event.get('plant_area')
                map['standard'] = event.get('standard')
                map['shelf_state'] = event.get('shelf_state')
                map['type'] = event.get('type')
     						#这里是关键，areaListTemp是临时集合，保存去重的数据，然后遍历到areaList中
                map['areaListTemp'] ||= []
                map['areaList'] ||= []
                if(event.get('area_id') != nil)
                    if !(map['areaListTemp'].include? event.get('area_id'))
                        map['areaListTemp'] << event.get('area_id')
                        map['areaList'] << {
                            'area_id' => event.get('area_id')
                        }
                    end
                end
                map['labelList'] ||= []
                map['goodsLabelList'] ||= []
                if(event.get('dictionary_value') != nil)
                    if !(map['labelList'].include? event.get('dictionary_value'))
                        map['labelList'] << event.get('dictionary_value')
                        map['goodsLabelList'] << {
                            'dictionary_value' => event.get('dictionary_value')
                        }
                    end
                end
                map['attributeList'] ||= []
                map['attributeValueList'] ||= []
                if(event.get('attribute_id') != nil)
                    if !(map['attributeList'].include? event.get('attribute_id'))
                        map['attributeList'] << event.get('attribute_id')
                        map['attributeValueList'] << {
                            'attribute_id' => event.get('attribute_id'),
                            'attribute_value' => event.get('attribute_value')
                        }
                    end
                end
                map['cateList'] ||= []
                map['categoryList'] ||= []
                if(event.get('category_id') != nil)
                    if !(map['cateList'].include? event.get('category_id'))
                        map['cateList'] << event.get('category_id')
                        map['categoryList'] << {
                            'category_id' => event.get('category_id')
                        }
                    end
                end
            event.cancel()"
						#使用聚合插件
            push_previous_map_as_event => true
						#超时时间，如果不设置，logstash不知道什么时候会结束，会导致最后一条数据丢失。这里应该有一个结束条件
						#设置5秒是一个不严谨的办法
            timeout => 5
        }
				#这里删除保存数据的临时集合和生成的一些默认的字段
        mutate  {
            remove_field => ["@version","labelList","attributeList","cateList","areaListTemp"]
            
    }
}
```

#### **踩坑记录**

关于数据的聚合，这里我查找了很多资料，试了很多的写法，始终有问题要不是数据会有丢失，要不会出现数据的错乱的情况。这里有几个地方需要注意下

- `task_id` 是sql查询的`id`，相当于每一个id是一个`task`，正常我们使用联表查询的时候，因为一对多的关系，会生成多条记录，`areaListTemp`保存了同一个id的多条数据中的`label`字段的值，并且进行去重， 如果`id`不是聚集在一起，可能导致临时的集合还没有保存完数据就被删除，导致数据的丢失。

- logstash会使用多线程进行聚合任务，如果同一个聚合任务被多个线程分隔操作，最后聚合的过程中可能会丢失数据，这里配置pipeline.yml文件，设置工作线程为1。（这里可能出现性能问题）

	```yaml
	pipeline.workers: 1
	```

- 多个数据源同时输入也有有坑，我这里需要维护两个`index`，所以需要使用两个`jdbc`，搜索网上的资料都是在一个`conf`文件中写，然后通过`type`去区别不同的数据源分别处理,，类似下面的处理方法

	```json
	input {
	  jdbc {
	  	#这里指定connector的位置
	  	jdbc_driver_library => "../mysql-connector-java-5.1.43-bin.jar"
	    ....
	  	type => goods
	    statement => "
	                SELECT .... and t1.last_update_time > :sql_last_value and t1.last_update_time < NOW() 			AND t1.is_delete=0 AND t1.type=1 order by t1.id desc"
		}
		jdbc {
	    ...
	    type => category
	    ...
	  }
	}
	output {
	  //这里根据上面配置的type进行不同的处理
	  if [type] == "goods" {
	     elasticsearch {
					hosts => ["localhost: 9200"]   
					index => "goods"
					document_id => "%{id}"
	  	}
	  }
		if [type] == "category" {
	     elasticsearch {
					hosts => ["localhost: 9200"]   
					index => "category"
					document_id => "%{id}"
	  	}
	  }
	}
	```

	但是我在7.6的版本中按照这样的格式每次只能生成一个`index`，也没有报错，后来我配置了两个`conf`文件，然后在`pineline.yml`中配置多个通道，进行处理

	```yaml
	- pipeline.id: goods
	  path.config: "../config/goods.conf"
	  pipeline.workers: 1
	  # pipeline.batch.size: 1000
	  # pipeline.output.workers: 3
	  # queue.type: persisted
	
	- pipeline.id: category
	  path.config: "../config/category.conf"
	```

	每个`pineline`对应一个`conf`文件的解析，终于解决了问题。

	

- 对于嵌套类型的数据结构，需要首先在`elasticsearch`中创建好`index`的`mapping`，否则`logstash`不能自动识别。具体`mapping`格式如下

	```json
	{
	    "mappings": {
	        "properties": {
	            "address": {
	                "type": "text",
	                "fields": {
	                    "keyword": {
	                        "type": "keyword",
	                        "ignore_above": 256
	                    }
	                }
	            },
	            "areaList": {
	                "type": "nested",
	                "properties": {
	                    "area_id": {
	                        "type": "text",
	                        "fields": {
	                            "keyword": {
	                                "type": "keyword",
	                                "ignore_above": 256
	                            }
	                        }
	                    }
	                }
	            },
	            "area_id": {
	                "type": "text",
	                "fields": {
	                    "keyword": {
	                        "type": "keyword",
	                        "ignore_above": 256
	                    }
	                }
	            },
	            "attributeValueList": {
	                "type": "nested",
	                "properties": {
	                    "attribute_id": {
	                        "type": "text",
	                        "fields": {
	                            "keyword": {
	                                "type": "keyword",
	                                "ignore_above": 256
	                            }
	                        }
	                    },
	                    "attribute_value": {
	                        "type": "text",
	                        "fields": {
	                            "keyword": {
	                                "type": "keyword",
	                                "ignore_above": 256
	                            }
	                        }
	                    }
	                }
	            },
	            "categoryList": {
	                "type": "nested",
	                "properties": {
	                    "category_id": {
	                        "type": "text",
	                        "fields": {
	                            "keyword": {
	                                "type": "keyword",
	                                "ignore_above": 256
	                            }
	                        }
	                    }
	                }
	            },
	            "company_id": {
	                "type": "text",
	                "fields": {
	                    "keyword": {
	                        "type": "keyword",
	                        "ignore_above": 256
	                    }
	                }
	            },
	            "company_name": {
	                "type": "text",
	                "fields": {
	                    "keyword": {
	                        "type": "keyword",
	                        "ignore_above": 256
	                    }
	                }
	            },
	            "description": {
	                "type": "text",
	                "fields": {
	                    "keyword": {
	                        "type": "keyword",
	                        "ignore_above": 256
	                    }
	                }
	            },
	            "goodsLabelList": {
	                "type": "nested",
	                "properties": {
	                    "dictionary_value": {
	                        "type": "text",
	                        "fields": {
	                            "keyword": {
	                                "type": "keyword",
	                                "ignore_above": 256
	                            }
	                        }
	                    }
	                }
	            },
	            "id": {
	                "type": "text",
	                "fields": {
	                    "keyword": {
	                        "type": "keyword",
	                        "ignore_above": 256
	                    }
	                }
	            },
	            "name": {
	                "type": "text",
	                "analyzer": "ik_max_word",
	                "fields": {
	                    "keyword": {
	                        "type": "keyword",
	                        "ignore_above": 256
	                    }
	                }
	            },
	            "number": {
	                "type": "text",
	                "fields": {
	                    "keyword": {
	                        "type": "keyword",
	                        "ignore_above": 256
	                    }
	                }
	            },
	            "plant_area": {
	                "type": "text",
	                "fields": {
	                    "keyword": {
	                        "type": "keyword",
	                        "ignore_above": 256
	                    }
	                }
	            },
	            "price": {
	                "type": "float",
	                "fields": {
	                    "keyword": {
	                        "type": "keyword",
	                        "ignore_above": 256
	                    }
	                }
	            },
	            "shelf_state": {
	                "type": "long"
	            },
	            "shelf_stock": {
	                "type": "text",
	                "fields": {
	                    "keyword": {
	                        "type": "keyword",
	                        "ignore_above": 256
	                    }
	                }
	            },
	            "shelf_time": {
	                "type": "date"
	            },
	            "sl_url": {
	                "type": "text",
	                "fields": {
	                    "keyword": {
	                        "type": "keyword",
	                        "ignore_above": 256
	                    }
	                }
	            },
	            "standard": {
	                "type": "text",
	                "fields": {
	                    "keyword": {
	                        "type": "keyword",
	                        "ignore_above": 256
	                    }
	                }
	            },
	            "stock_info": {
	                "type": "text",
	                "fields": {
	                    "keyword": {
	                        "type": "keyword",
	                        "ignore_above": 256
	                    }
	                }
	            },
	            "tags": {
	                "type": "text",
	                "fields": {
	                    "keyword": {
	                        "type": "keyword",
	                        "ignore_above": 256
	                    }
	                }
	            },
	            "type": {
	                "type": "long"
	            }
	        }
	    }
	}
	```

	这里主要注意`areaList`中的`type`设置为`nested`，如果需要使用分词器的话，也可以设置好，例如`name`字段使用了一个比较好用的中文`ik`分词器。