---
title: "Go优雅关闭http服务"
date: "2022-05-09"
categories:
  - Golang
toc: true
---

**优雅退出**：指HTTP服务在接到用户的退出指令后，停止接受新请求，对进行中的请求处理完成后再退出。

<!--more-->

如下代码，`main`中启动`http.ListenAndServe`，在`goroutine`中`signal.Notify`监听退出信号，接口里sleep n秒模拟请求处理中。

测试：本地请求`curl http://localhost:8080`后，马上`Ctrl + C`。分别在处理函数里 sleep 3秒、8秒、20秒。看看区别。

这里有一个需要注意的点是，`Shutdown`在调用后，`ListenAndServe`立马返回，如果我们把`ListenAndServe`放在主函数`main`里，程序直接退出，无法等待`Shutdown`执行完成，参考[Golang http.Server安全退出：容易被误用的Shutdown()方法](https://zhuanlan.zhihu.com/p/413070530) 

```go
package main

import (
	"context"
	"log"
	"net/http"
	"os"
	"os/signal"
	"syscall"
	"time"
)

type Engine struct{}

func (engine *Engine) ServeHTTP(w http.ResponseWriter, req *http.Request) {
	time.Sleep(time.Second * 20)
	w.WriteHeader(200)
	w.Write([]byte("ok"))
}

func main() {
	engine := new(Engine)
	server := &http.Server{
		Addr:         ":8080",
		Handler:      engine,
		ReadTimeout:  5 * time.Second,
		WriteTimeout: 5 * time.Second,
	}

	// http服务
	go func() {
		log.Println("HTTP服务启动", "http://localhost"+server.Addr)
		if err := server.ListenAndServe(); err != nil && err != http.ErrServerClosed {
			log.Println(err)
			os.Exit(0)
		}
		log.Println("HTTP服务关闭请求")
	}()

	// 监听信号 优雅退出http服务
	Watch(func() error {
		ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
		defer cancel()
		return server.Shutdown(ctx)
	})

	log.Println("程序退出")
}

// 监听信号
func Watch(fns ...func() error) {
	// 程序无法捕获信号 SIGKILL 和 SIGSTOP （终止和暂停进程），因此 os/signal 包对这两个信号无效。
	ch := make(chan os.Signal, 1)
	signal.Notify(ch, syscall.SIGHUP, syscall.SIGINT, syscall.SIGTERM, syscall.SIGQUIT, syscall.SIGUSR1, syscall.SIGUSR2)

	// 阻塞
	s := <-ch
	close(ch)
	log.Println("接收到信号", s.String(), "执行关闭函数")
	for i := range fns {
		if err := fns[i](); err != nil {
			log.Println(err)
		}
	}
	log.Println("关闭函数执行完成")
}

```
