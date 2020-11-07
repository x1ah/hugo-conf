+++
title = "长连接聊天室 Demo"
date = "2020-11-03"
author = "x1ah"
tags = ["HTTP", "Go"]
keywords = ["HTTP", "Go"]
description = "一个长连接实现 IM 功能的小 demo"
showFullContent = false
+++


#### Server

```go
package main

import (
    "bufio"
    "fmt"
    "net"
)

// 用来记录所有的客户端连接
var ConnMap map[string]*net.TCPConn

func main() {
    var tcpAddr *net.TCPAddr
    ConnMap = make(map[string]*net.TCPConn)
    tcpAddr, _ = net.ResolveTCPAddr("tcp", "127.0.0.1:9999")

    tcpListener, _ := net.ListenTCP("tcp", tcpAddr)

    defer tcpListener.Close()

    for {
        tcpConn, err := tcpListener.AcceptTCP()
        if err != nil {
            continue
        }

        fmt.Println("A client connected : " + tcpConn.RemoteAddr().String())
        // 新连接加入map
        ConnMap[tcpConn.RemoteAddr().String()] = tcpConn
        go tcpPipe(tcpConn)
    }

}

func tcpPipe(conn *net.TCPConn) {
    ipStr := conn.RemoteAddr().String()
    defer func() {
        fmt.Println("disconnected :" + ipStr)
        conn.Close()
    }()
    reader := bufio.NewReader(conn)

    for {
        message, err := reader.ReadString('\n')
        if err != nil {
            return
        }
        fmt.Println(conn.RemoteAddr().String() + ":" + string(message))
        // 这里返回消息改为了广播
        boradcastMessage(conn.RemoteAddr().String() + ":" + string(message))
    }
}


func boradcastMessage(message string) {
    b := []byte(message)
    // 遍历所有客户端并发送消息
    for _, conn := range ConnMap {
        conn.Write(b)
    }
}
```


#### Client

```go
package main

import (
    "bufio"
    "fmt"
    "net"
)

func main() {
    var tcpAddr *net.TCPAddr
    tcpAddr, _ = net.ResolveTCPAddr("tcp", "127.0.0.1:9999")

    conn, _ := net.DialTCP("tcp", nil, tcpAddr)
    defer conn.Close()
    fmt.Println("connected!")

    go onMessageRecived(conn)

    // 控制台聊天功能加入
    for {
        var msg string
        fmt.Scanln(&msg)
        if msg == "quit" {
            break
        }
        b := []byte(msg + "\n")
        conn.Write(b)
    }
}

func onMessageRecived(conn *net.TCPConn) {
    reader := bufio.NewReader(conn)
    for {
        msg, err := reader.ReadString('\n')
        fmt.Println(msg)
        if err != nil {
            break
        }
    }
}
```

#### Run

```shell
> go run server.go
```

```shell
> go run client.go
```
