---
layout: post
title:  Golang标准库net/rpc导致程序崩溃？
date:   2020-02-15 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: post-1.jpg # Add image post (optional)
tags: [Blog, Golang, net/rpc]
author: zhongjin616
---

2020年春节。公司的业务在春节期间流量猛增，运维报告发现有容器重启的现象。
拿到日志，根据堆栈信息的提示，查看出错的代码行, 第一感觉是这里怎么会出错？gorutine里定义的局部变量肯定只有它自己能访问到。这里贴一下我用于复现问题而写的代码示例。


``` client/main.go
package main

import (
	"fmt"
	"net/rpc"
	"time"
)

type Args struct {
	A, B int
}

type Result struct {
	A, B     int
	Quotient map[string]int
}

const N = 1000 // 起1000个goroutine同时进行rpc请求

func main() {
	for i := 0; i < N; i++ {
		go doRPC(i)
	}
	select {}
}

func doRPC(id int) {
	c, err := rpc.Dial("tcp", ":8080")
	if err != nil {
		fmt.Println("dial: ", err)
		return
	}

	arg := Args{10, 3}
	res := new(Result)
	sm := "Arith.Divide"
	select {
	case resCall := <- c.Go(sm, arg, res, nil).Done:
		if resCall.Error != nil {
			fmt.Println("rpc: ", resCall.Error)
		} else {
			fmt.Println(id, " do rpc successfully")
		}
		c.Close()
	case <-time.After(time.Millisecond):
		fmt.Println(id, " do rpc timeout")
		c.Close() // timeout后关闭client
	}

	if len(res.Quotient) == 0 { // race may occure
		for k, v := range res.Quotient {
			fmt.Println("k: ", k, " v: ", v)
		}
	}
	return
}

```

``` server/main.go

package main

import (
	"errors"
	"fmt"
	"net"
	"net/rpc"
)

type Args struct {
	A, B int
}

type Result struct {
	A, B     int
	Quotient map[string]int
}

type Arith int

func (t *Arith) Divide(args *Args, reply *Result) error {
	if args.B == 0 {
		return errors.New("divide by zero")
	}
	reply.A = args.A
	reply.B = args.B
	reply.Quotient = map[string]int{
		"Quo": args.A / args.B,
		"Rem": args.A % args.B,
	}
	return nil
}

func main() {
	arith := new(Arith)
	rpc.Register(arith)
	ln, err := net.Listen("tcp", ":8080")
	if err != nil {
		fmt.Println("liseten error: ", err)
	}

	fmt.Println("start server listening 8080")
	cnt := 0
	for {
		conn, err := ln.Accept()
		if err != nil {
			fmt.Println("accept: ", err)
			conn.Close()
			continue
		}
		fmt.Println("serve conn: ", cnt)
		go rpc.ServeConn(conn)
		cnt++
	}
}

```

虽然一时找不到方向，但还是得坚信Go语言本身没毛病，是咱自己的问题。那么只能去搞清楚net/rpc的异步请求到底是怎么处理的。我们是把局部变量的地址传了进去, 所以还是有可能出现问题的。
只好去看rpc的源代码。在net/rpc/client.go中，NewClient函数里，可是把Client.input放到一个goroutine里单独执行了，那么这样的确是会出现goroutine竞争的情况。

启动server。在client中，我们使用go run --race main.go启动，这样容易出现race的现象。

```
797  do rpc timeout
==================
WARNING: DATA RACE
Read at 0x00c00000c2f0 by goroutine 238:
  main.doRPC()
      /home/f2/go/src/sandbox/netrpc/client/main.go:50 +0x311

Previous write at 0x00c00000c2f0 by goroutine 246:
  [failed to restore the stack]

Goroutine 238 (running) created at:
  main.main()
      /home/f2/go/src/sandbox/netrpc/client/main.go:22 +0x50

Goroutine 246 (finished) created at:
  net/rpc.NewClientWithCodec()
      /usr/local/go/src/net/rpc/client.go:206 +0x10d
  net/rpc.NewClient()
      /usr/local/go/src/net/rpc/client.go:196 +0x30d
  net/rpc.Dial()
      /usr/local/go/src/net/rpc/client.go:279 +0xe7
  main.doRPC()
      /home/f2/go/src/sandbox/netrpc/client/main.go:28 +0x70
==================
```

这里还有一个问题值得注意。虽然在timeout之后，我们调用了c.Close()对client进行了处理，
还是出现了同时读写map的问题。从代码中看，Close是关闭了连接，这时再读数据肯定就是出错。所以应该是数据已经接收完毕，gob正在做反序列化的阶段了。
