## 搭建日志系统EFK

> 使用docker 快速构建kibana , elasticsearch (此处查看docker快速构建) filebeat

> filebeat 主要用于收集日志

```
//下载filebeat
curl -L -O https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.10.1-darwin-x86_64.tar.gz
tar xzvf filebeat-7.10.1-darwin-x86_64.tar.gz
cd filebeat-7.10.1-darwin-x86_64/

//修改配置
vim filebeat.yml

output.elasticsearch:
  hosts: ["<es_url>"]
  username: "elastic"
  password: "<password>"
setup.kibana:
  host: "<kibana_url>"

//开启配置
./filebeat modules enable system

//配置索引
./filebeat setup

//启动
./filebeat -e
```

> golang 日志写入es

```
//go引入拓展包
"github.com/olivere/elastic/v7"
"github.com/sirupsen/logrus"
"gopkg.in/sohlich/elogrus.v7"

//日志格式
跟踪，调试，信息，警告，错误，严重, 紧急
logrus.Trace("Something very low level.")
logrus.Debug("Useful debugging information.")
logrus.Info("Something noteworthy happened!")
logrus.Warn("You should probably take a look at this.")
logrus.Error("Something failed but I'm not quitting.")
logrus.Fatal("Bye.")
logrus.Panic("I'm bailing.")

//链接es
log := logrus.New()
client, err := elastic.NewClient(elastic.SetSniff(false),elastic.SetURL("http://192.168.59.131:9200"))
if err != nil {
    log.Panic(err)
}
//设置日志命中索引
hook, err := elogrus.NewAsyncElasticHook(client, "localhost", logrus.DebugLevel, "mylog")
if err != nil {
    log.Panic(err)
}
log.Hooks.Add(hook)

//打印日志
log.Debug("test log.Debug")
```
