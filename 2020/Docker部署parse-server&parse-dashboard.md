## 创建mongo

```
docker pull mongo:latest
docker run -itd --name mongo -p 27017:27017 mongo --auth
```

### 创建mongo数据库的账号和密码
```
docker exec -it mongo mongo admin
use admin
# 创建一个名为 admin，密码为 123456 的用户。
>  db.createUser({ user:'admin',pwd:'123456',roles:[ { role:'userAdminAnyDatabase', db: 'admin'},"readWriteAnyDatabase"]});
# 尝试使用上面创建的用户信息进行连接。
> db.auth('admin', '123456')
use apps
db.createUser({user:"root",pwd:"admin",roles:[{role:"readWrite",db:"apps"}]});
db.auth("root","admin")
```

## 创建parse-server
```
docker run --name parse_server  -p 1337:1337 --link mongo:mongo -d parseplatform/parse-server --appId myapp --masterKey appKey --databaseURI mongodb://mongo/app
```

* 创建时要给parse指定一个appId和masterKey

* mongodb://mongo/app   #app为创建的数据库的名字


## 创建parse-dashboard

```
docker run --name parse_dashboard -d -p 8080:4040 -v /Users/antarts/javascriptbridge/parse-dashboard/Parse-Dashboard/parse-dashboard-config.json:/src/Parse-Dashboard/parse-dashboard-config.json parseplatform/parse-dashboard --allowInsecureHTTP
```

* --allowInsecureHTTP #parse_dashboard默认只能用https访问，此参数可以开启http访问

### parse-dashboard-config.json内容
``` javascript
{
  "apps": [{
    "serverURL": "https://ip:1337/parse",      #parse-server服务的域名
    "appId": "myapp",                               
    "masterKey":"appKey,                          #parse-server的ID及Key
    "appName": "apps"                             #app的名字
  }],
  "users":[{
    "user":"test",
    "pass":"testapp"                             #开启账号密码登录的参数(可以开启多个账号)
   }]
  
}
```

浏览器打开http://ip:8080 进入dashboard界面


---

> 参考文献
1. [docker部署parse-server和parse-dashboard](https://blog.csdn.net/qq_39965424/article/details/106900620)
2. [parse-dashboard
](https://hub.docker.com/r/parseplatform/parse-dashboard)
3. [parse-server
](https://hub.docker.com/r/parseplatform/parse-server)
4. [Docker 安装 MongoDB](https://www.runoob.com/docker/docker-install-mongodb.html)