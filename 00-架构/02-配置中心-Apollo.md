## Apollo

#### 简介

Apollo 是协程框架部门研发的开源配置管理中心，能够集中化管理应用 不同环境、不同集群的配置。

配置修改后，能够实时推送给应用端。

支持多种客户端：`Java`、 `.Net`、 `Go`、 `Python`、 `NodeJS`、 `PHP`。 

安装文档：https://github.com/ctripcorp/apollo/wiki/Quick-Start

官方文档：https://github.com/ctripcorp/apollo/wiki

#### 核心参数

- AppID：这个是创建之后生成的。
- Cluster：选择某个集群的配置项。 
- NameSpaceNames：配置项的集合，默认为：application。
- CacheDir：本地的缓存配置项的目录。
- IP：不同的环境配置 IP，比如，测试环境，预上线环境，发布环境。

#### Go 客户端

agollo is a golang client for ctrip apollo config center

链接：https://github.com/philchia/agollo

案例：项目配置如下

```Go
type Config struct {
	API_ADDRESS string `apollo:"API_ADDRESS" namespace:"application" default:""`
}
```

当项目启动时：

1、启动 Apollo ：

```go
func ApolloStart() {
  var AppID     = ""
  var Cluster   = ""
  var NameSpace = ""
  var CacheDir  = ""
  var IP        = ""
  config := agollo.Conf{
    AppID          : AppID, 
    Cluster        : Cluster, 
    NameSpaceNames : NameSpace, 
    CacheDir       : CacheDir, 
    IP             : IP
  }
	agollo.StartWithConf(&config)
}
```



2、拉取 Apollo 配置：

```go
func InitApolloConfig() {
  conf := Config
  v := reflect.ValueOf(conf).Elem()
	for i := 0; i < v.NumField(); i++ {
		fieldInfo := v.Type().Field(i)
    
		apolloKey       := fieldInfo.Tag.Get("apollo")
		apolloNamespace := fieldInfo.Tag.Get("namespace")
		defaultValue    := fieldInfo.Tag.Get("default")
    
		val := agollo.GetStringValueWithNameSpace(apolloNamespace, apolloKey, defaultValue)
		v.FieldByName(string(fieldInfo.Name)).Set(reflect.ValueOf(val))
	}
}
```



3、订阅 Apollo 更新：

```Go
func SubscribeToUpdate() {
  for _, v := range NameSpaceNames {
		agollo.SubscribeToNamespaces(v)
	}
	events := agollo.WatchUpdate()
	for {
		changeEvent := <-events
    // 更新本地配置
		updateLocalConfig(changeEvent)
	}
}

func updateLocalConfig(event *agollo.ChangeEvent) {
  conf := Config
  for apollo_k, apollo_v: range event.Changes {
    v := reflect.ValueOf(conf).Elem()
    for i := 0; i < v.NumField(); i++ {
      fieldInfo := v.Type().Field(i)
      
      apolloKey       := fieldInfo.Tag.Get("apollo")
		  apolloNamespace := fieldInfo.Tag.Get("namespace")
      
      if event.Namespace == apolloNamespace && apollo_k == apolloKey {
        v.FieldByName(string(fieldInfo.Name)).Set(reflect.ValueOf(apollo_v.NewValue))
      }
    }
  }
}
```

