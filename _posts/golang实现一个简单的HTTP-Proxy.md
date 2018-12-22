---
title: golang实现一个简单的HTTP-Proxy
data: '2018-08-05 13:00'
abbrlink: fc01
date: 2018-08-05 13:32:19
tags:
	- Golang
	- HTTP
---


最近学习Golang，为了学而学没什么意思，于是自己写了一个简单的HTTP代理,总共代码100行左右。
不得不说用Go写就是爽，来了一个Conn直接开Goroutine去处理，也不需要像Java那种小心翼翼的控制线程，
为了高并发还要用Select,Poll,Epoll。
<!--more-->
```golang
package main

import (
    "bufio"
    "fmt"
    "io"
    "log"
    "net"
    "net/http"
    "os"
)

const Read_Size = 1024 * 2

type readerOnly struct {
    io.Reader
}

type server struct {
    outWriter io.Writer
    errWriter io.Writer
}

func (s *server) handlePlainHTTPRequest(conn net.Conn, request *http.Request) {
    rawHost := request.Host + ":80"
    server, err := net.Dial("tcp", rawHost)
    if err != nil {
        fmt.Fprintf(s.errWriter, "Error when connect to %s\n", rawHost)
        s.Close(conn, nil)
        return
    }
    request.Write(server)
    s.sideToAnotherSide(conn, server)
    s.sideToAnotherSide(server, conn)
}

func (s *server) handleRequest(conn net.Conn) {
    reader := bufio.NewReaderSize(readerOnly{conn}, Read_Size)
    request, err := http.ReadRequest(reader)

    if err != nil {
        fmt.Errorf(err.Error() + "\n")
        return
    }

    if request.Method != "CONNECT" {
        s.handlePlainHTTPRequest(conn, request)
        return
    }

    rawHost := request.Host

    conn.Write([]byte("HTTP/1.1 200 Connection Established\r\n\r\n"))

    server, err := net.Dial("tcp", rawHost)

    s.sideToAnotherSide(conn, server)
    s.sideToAnotherSide(server, conn)

}

func (s *server) sideToAnotherSide(side net.Conn, another net.Conn) {
    go func(side net.Conn, another net.Conn) {
        for {
            buf := make([]byte, Read_Size)
            count, err := side.Read(buf)
            if err != nil || count <= 0 {
                fmt.Fprintf(s.errWriter, "Error %s\n", err.Error())
                s.Close(side, another)
                return
            }
            num, err := another.Write(buf[:count])
            if err != nil || num <= 0 {
                s.Close(side, another)
                return
            }
        }

    }(side, another)
}

func (s *server) Close(local net.Conn, server net.Conn) {

    if local != nil {
        local.Close()
    }

    if server != nil {
        server.Close()
    }

}

func main() {
    l, err := net.Listen("tcp", ":9000")
    if err != nil {
        log.Panic(err)
    }

    for {
        conn, error := l.Accept()
        fmt.Fprintf(os.Stdout, "Recv connection from: %s\n", conn.RemoteAddr())
        if error != nil {
            log.Panic(error)
        }
        s := &server{os.Stdout, os.Stderr}
        go s.handleRequest(conn)
    }

}
```
