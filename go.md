# 全局配置

```sh
tee -a ~/.zshrc <<EOF
## Golang
#export GOROOT="/Users/lan/Landercy/Workspace/apps/go115"
export GOROOT="/home/lan/Workspace/apps/go116"
export PATH=\$PATH:"\$GOROOT/bin"

#export GOPATH="/Users/lan/Landercy/Workspace/apps/gopath"
export GOPATH="/home/lan/Workspace/apps/gopath"
export GOBIN="\$GOPATH/bin"
export PATH=\$PATH:\$GOBIN
export GO111MODULE=on
export GOPROXY=https://mirrors.aliyun.com/goproxy,https://goproxy.io,https://goproxy.cn,direct
## END Golang
EOF
source ~/.zshrc
```

# Go Mod

```sh
# 初始化项目
mkdir idea
cd idea
go mod init idea

# 查看可用版本
go list -m -versions github.com/astaxie/beego
go list -m -versions github.com/beego/bee

# 安装依赖
go get -v -u github.com/astaxie/beego@v1.12.0
go get -v -u github.com/beego/bee@v1.12.0

# 更换依赖包版本
go mod edit -require="github.com/gin-gonic/gin@v1.3.0"

# 更新依赖
go mod tidy 
```

```sh
# 查看所有依赖包
go list -m all
```

# Go Mod寻包机制

```go
import (
	"github.com/gin-gonic/gin"
)
```

不使用go mod时，
go build 会从GOROOT和GOPATH的src目录下寻找包

> main.go:4:2: cannot find package "github.com/gin-gonic/gin" in any of:
> /usr/local/Cellar/go/1.13.5/libexec/src/github.com/gin-gonic/gin (from `$GOROOT)
> 	/Users/mac/Raymond/Workspace/projects/go/gopath/src/github.com/gin-gonic/gin (from $`GOPATH)

使用go mod时，go build会下载依赖包到GOPATH/pkg/mod目录下

```sh
bee new <project_name>
```

### 打包命令

```sh
CGO_ENABLED=0 GOARCH=amd64 GOOS=linux  go build -o dist/linux
CGO_ENABLED=0 GOARCH=amd64 GOOS=windows go build -o dist/windows
CGO_ENABLED=0 GOARCH=amd64 GOOS=darwin go build -o dist/mac

cp ./config.yaml dist/
```

```sh
nohup ./canary > "log.$(date +%m%d%H%M).txt" 2>&1  &
```

## robotgo 打包windows

> <https://pkgs.org/download/mingw64-gcc>

```sh
CGO_ENABLED=1 GOARCH=amd64 GOOS=windows CC=x86_64-w64-mingw32-gcc CXX=x86_64-w64-mingw32-g++ go build -x ./
```


  
# ENV
```sh
# GoLang
export GOPATH="/workspace/code/go"
export GOROOT="/workspace/apps/go"
export PATH="$PATH:$GOROOT/bin"
export CGO_ENABLE=1
export GO111MODULE=on
export GOPROXY=https://mirrors.aliyun.com/goproxy/,https://goproxy.io,https://goproxy.cn,direct
```

# RUN
```sh
pwd
go install
go build
ps -e | grep "canary"| awk '{print $1}'|xargs -r kill -9 && echo 'Killed'
rm -f /data/web/_canary/canary
cp ./canary /data/web/_canary/canary
cd /data/web/_canary
nohup ./canary > "log.$(date +%m%d%H%M).txt" 2>&1  &
```



# redigo

```go
package main

import (
	"fmt"
	"github.com/gomodule/redigo/redis"
)

func main() {
	c, err := redis.Dial("tcp", "centos:6379")
	if err != nil {
		fmt.Println("Connect to redis error", err)
	}
	defer c.Close()

	_, err = c.Do("SET", "redigo", "v")
	if err != nil {
		fmt.Println("redis set failed", err)
	}

	// 需要转换类型
	val1, err := redis.String(c.Do("GET", "redigo"))
	if err != nil {
		fmt.Println("redis get failed", err)
	} else {
		fmt.Println("get key", val1)
	}
}
```

# go-redis

```go
package main

import (
	"fmt"
	"github.com/go-redis/redis"
	"time"
)

func main() {
	// connect
	client := redis.NewClient(&redis.Options{
		Addr: "centos:6379",
	})

	// close
	defer client.Close()

	// ping
	pong, err := client.Ping().Result()
	fmt.Println(pong, err)

	// set
	err = client.Set("string", "v", 0).Err()
	client.Set("string1", "v", 20*time.Second)

	// expire
	client.Expire("string", 10*time.Second)

	// get
	val, err := client.Get("string").Result()
	fmt.Println("string", val)

	// get2
	val2, err := client.Get("string2").Result()
	if err == redis.Nil {
		fmt.Println("string2 does not exists")
	} else if err != nil {
		panic(err)
	} else {
		fmt.Println("string2", val2)
	}

	// hset, hmset
	client.HSet("hashset", "f1", "v1")
	client.HMSet("hashset", "f2", "v2", "f3", "v3")

	// hget
	val3, err := client.HGet("hashset", "f1").Result()
	fmt.Println("hashset.f1", val3)

	// hmget
	val4, err := client.HMGet("hashset", "f1", "f3").Result()
	fmt.Println("hashset", val4)

	// hgetall
	val5, err := client.HGetAll("hashset").Result()
	fmt.Println("hashset.f1", val5)

	// push
	client.LPush("list", 1)
	client.LPush("list", 2)
	client.RPush("list", 3)
	client.RPush("list", 4)
	client.LPop("list")
	client.RPop("list")

	// len
	len, err := client.LLen("list").Result()
	fmt.Println("list.len", len)

	// range
	val6, err := client.LRange("list", 0, len).Result()
	fmt.Println("list", val6)

}
```

# GORM

## 连接

```go
package main

import (
    "github.com/jinzhu/gorm"
   // 导入MySQL驱动
    _ "github.com/jinzhu/gorm/dialects/mysql"
)

func main() {
  db, err := gorm.Open("mysql", "user:password@tcp(host:port)/dbname?charset=utf8&parseTime=True&loc=Local")
  defer db.Close()
}
```

## 自动迁移

> 自动迁移仅仅会创建表，缺少列和索引，并且不会改变现有列的类型或删除未使用的列以保护数据。

```go
db.AutoMigrate(&User{})

db.AutoMigrate(&User{}, &Product{}, &Order{})

// 创建表时添加表后缀
db.Set("gorm:table_options", "ENGINE=InnoDB").AutoMigrate(&User{})
```

> 报错的问题
>
>  sql: Scan error on column index 1, name "created_at": unsupported Scan, storing driver.Value type []uint8 into type *time.Time 
>
>  root:123123@tcp(localhost:3306)/linde_auth?charset=utf8&parseTime=True
>
> 加上 parseTime=True


```go
package main

import (
	"fmt"
	"github.com/jinzhu/gorm"
	_ "github.com/jinzhu/gorm/dialects/mysql"
)

type User struct {
	Id   int
	Name string
}

func main() {
	db, err := gorm.Open("mysql", "root:123123@tcp(centos:3306)/test?charset=utf8")
	if err != nil {
		panic("failed to connect database")
	}
	defer db.Close()

	// insert
	user := User{Name: "test008"}
	db.Create(&user)
	fmt.Println("InsertId:", user.Id)

	// update
	user.Name = "test008889"
	db.Save(user)

	// delete
	db.Delete(&user)

	// select
	user = User{}
	db.First(&user, 1)
	fmt.Println(user)

	user = User{}
	db.Where("name = ?", "test008").First(&user)
	fmt.Println(user)

	user = User{}
	db.Where(&User{Name: "test008"}).First(&user)
	fmt.Println(user)

	user = User{}
	db.Where(map[string]interface{}{"name": "test008"}).First(&user)
	fmt.Println(user)

	users := []User{}
	db.Where([]int{2, 5, 8}).Find(&users)
	fmt.Println(users)

}
```


# go sql driver
```go
package main

import (
	"database/sql"
	"fmt"
	_ "github.com/go-sql-driver/mysql"
)

func main() {
	// connect
	db, err := sql.Open("mysql", "root:123123@tcp(centos)/test?charset=utf8")
	if err != nil {
		panic(err)
	}
	defer db.Close()

	var stmt *sql.Stmt
	var res sql.Result
	var insertId int64
	var affectRows int64

	// insert
	//stmt, err = db.Prepare("insert beego values(?,?)")
	stmt, err = db.Prepare("insert beego set id = ? , name = ?")
	res, err = stmt.Exec(nil, "test004")
	insertId, err = res.LastInsertId()
	fmt.Println("LastInsertId:", insertId)

	// update
	stmt, err = db.Prepare("update beego set name = ? where id = ?")
	res, err = stmt.Exec("test005", 5)
	affectRows, err = res.RowsAffected()
	fmt.Println("AffectRows:", affectRows)

	// delete
	stmt, err = db.Prepare("delete from beego where id > ?")
	res, err = stmt.Exec(5)
	affectRows, err = res.RowsAffected()
	fmt.Println("AffectRows:", affectRows)

	// query
	rows, err := db.Query("select * from beego")

	var id int
	var name string

	for (rows.Next()) {
		rows.Scan(&id, &name)
		fmt.Println(id, name)
	}
}
```