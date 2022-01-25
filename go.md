# go 学习笔记

第六版 下 预览

span里面有obj，一共67种class， class 0 就是一个大内存

大块内存=span 

 
# http

* 分hostname， port：

```
u, err := url.Parse("http://localhost:8080")
host, port, err := net.SplitHostPort(u.Host)
```

* http client

```
transport.go    (t *Transport) dialCon

readloop:

 _, err := pc.br.Peek(1) --这里只看一下，没数据也没事，eof也没事
 
rc := <-pc.reqch      reqch     chan requestAndChan // written by roundTrip; read by readLoop  

pc.readResponse

```

所以就是read先等write信号，write完，跑read

对于eof pc.readResponse`挂了

```
   2172 ▸   ▸   resp, err = ReadResponse(pc.br, rc.req)¬
|   2173 ▸   ▸   if err != nil {¬
|   2174 ▸   ▸   ▸   return¬
|   2175 ▸   ▸   }¬
```

应该是这里返回eof了


# test & gin

* go-sqlmock + gin : https://blog.csdn.net/vvstormhao/article/details/112376130
 * SkipInitializeWithVersion 需要设置为true
 * 	pool.SetMaxIdleConns(3) pool.SetMaxOpenConns(3) 这个不能设置否则跑不通
* gin 里面传递数据库信息： https://github.com/gin-gonic/gin/issues/932#issuecomment-306242400
* gin 优雅关闭 https://github.com/valord577/webdav/blob/main/cmd/serv.go#L69
* 用group的话， 要把最后留出来，group里类似 GET("/")就不好使了
* setup, teardown: https://geektutu.com/post/quick-go-test.html
* time.Now() or time.Now().UTC() 没必要调用utc https://stackoverflow.com/questions/44873825/how-to-get-timestamp-of-utc-time-with-golang
* 跳过case：  t.Skip("Skipping testing in CI environment")
* 判断是否在go test中：https://stackoverflow.com/questions/14249217/how-do-i-know-im-running-within-go-test
* 传递参数/系统变量给gotest，参数没成功，变量可以：https://siongui.github.io/2017/04/28/command-line-argument-in-golang-test/
* go test -timeout 99999s
* func Short() bool   Short reports whether the -test.short flag is set.
* 不lint某文件:https://stackoverflow.com/questions/65985835/how-skip-file-in-golangci-lint vim ale重启生效

# mysql

* 执行sql: https://stackoverflow.com/questions/49545146/how-to-exec-sql-file-with-commands-in-golang/49550419
* 获得当前目录： https://stackoverflow.com/questions/23847003/golang-tests-and-working-directory
* 一个sql语句由于有``，所以在go支持很不好，解决办法是 双引号用+ 连接起来，因为双引号不能换号， ``里面的``不能转义。。。。
* 集成测试：go mysql mock的问题是需要指定sql去返回，不好。 用go containertest的问题是，无法给镜像跑程序。 最后用 https://github.com/ory/dockertest, 坑：
  * 连mysql需要建非root授权去连
  * mysql没有m1镜像，需要mysql/mysql-server. https://github.com/mysql/mysql-docker/blob/mysql-server/8.0/Dockerfile  

```go
	var db *sql.DB
	var err error

	pool, err := dockertest.NewPool("")
	pool.MaxWait = time.Minute * 5
	if err != nil {
		log.Fatalf("Could not connect to docker: %s", err)
	}

	resource, err := pool.Run("mysql/mysql-server", "8.0", []string{"MYSQL_ROOT_PASSWORD=123456"})
	if err != nil {
		log.Fatalf("Could not start resource: %s", err)
	}

	defer func() {
		if err := pool.Purge(resource); err != nil {
			log.Fatalf("Could not purge resource: %s", err)
		}
	}()

	fmt.Println("mysql docker exposed port is:" + resource.GetPort("3306/tcp"))
	if err := pool.Retry(func() error {
		var err error
		db, err = sql.Open("mysql", fmt.Sprintf("ut:123456@(localhost:%s)/live", resource.GetPort("3306/tcp")))
		if err != nil {
			spew.Dump(err)
			return err
		}
		//启动阶段不停的重试连接，如果连接上了，则需要授权，让宿主机能连上
		//一句话也不能少，因为要建立live的database
		resource.Exec([]string{"mysql", "-uroot", "-p123456", "-e", `CREATE USER 'ut'@'%' IDENTIFIED BY '123456';`}, dockertest.ExecOptions{})
		resource.Exec([]string{"mysql", "-uroot", "-p123456", "-e", `GRANT ALL ON *.* TO 'ut'@'%'`}, dockertest.ExecOptions{})
		resource.Exec([]string{"mysql", "-uroot", "-p123456", "-e", `flush privileges;`}, dockertest.ExecOptions{})
		resource.Exec([]string{"mysql", "-uroot", "-p123456", "-e", `create database live;`}, dockertest.ExecOptions{})
		return db.Ping()
	}); err != nil {
		log.Fatalf("Could not connect to database: %s", err)
	}

	// 在项目根目录执行:  ROOT=`pwd` go test ./...   -v -timeout 10000s
	execSql(db, os.Getenv("ROOT")+"/ddl/ddl.sql")
```




# ctx

* https://segmentfault.com/a/1190000022484275. 主携程cancel(), 子携程ctx.Done()
* 子goroutine的ctx也会收到Done(): `https://draveness.me/golang/docs/part3-runtime/ch06-concurrency/golang-context/`


# gin

* NopCloseReader的含义就是读完数据就没了，土的办法是read完之后，再造一个，好的方法，参见gin。。
* gin中间件读requestbody的正确方法：    
* 用shouldbindwithbody! ，但是如果用的话，你就都用withbody，别用json，仔细看代码，为什么  。。。。。。
* 多了个/ 导致307， 坑：https://github.com/gin-gonic/gin/issues/1004



# other

* signal需要缓存channel: https://studygolang.com/articles/23104
* logrus, text模式的fields是按字母顺序补的，在后面，有时看不到
* 获得 pid cmd.Process.Pid
* 找正则：https://colobu.com/2020/11/11/golang-regex-replace-example/  `FindStringSubmatch`





# go + gitlab

最大boss，私有gitlab使用非80，443端口，为什么不用80，443呢？因为如果小企业，电信出口不能暴露80，443。。。。

1. 你的项目go.mod必须要用域名来命名pacakge: xxx.com/group/project.git 
2. 项目只能import下级的目录 比如 import pkg/xxxxx
3. 同级目录引用，或引用上级，你在 pkg/yyy/yyy.go 需要 带域名引用 import xxx.com/group/project.git/pkg/xxx
4. 所以别用同级目录
5. 引用的project，增加deploy token，以便被引用。
6. 改gitconfig 
```
[url "https://TOKEN:PW@xxx.com:12345/group/project"]
    insteadOf = https://xxx.com/group/project

```
7. Enjoy




