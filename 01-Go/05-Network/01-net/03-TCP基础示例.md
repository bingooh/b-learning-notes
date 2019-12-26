## 示例
- echo server1
```
// telnet localhost 2000
func server(){
	l,_:=net.Listen("tcp",":2000")

	for{
		conn,err:=l.Accept()
		if err!=nil{log.Println(err);continue}

		go func(conn net.Conn) {
			io.Copy(conn,conn) //客户端关闭连接后会结束此方法
			conn.Close()       //关闭连接
		}(conn)
	}
}

func client(){
	conn,_:=net.Dial("tcp",":2000")
	
	go func(conn net.Conn) {
		var b bytes.Buffer
		io.Copy(b,conn)
		
		fmt.Println(b.Bytes()) //打印字节数组

		//bs,_:=ioutil.ReadAll(conn)
	}(conn)

	conn.Write([]byte{1,2,3})

    //等待1秒后关闭
    //如果直接关闭可能来不及接收服务端返回的数据
    //如果不关闭，则无法结束客户端的io.Copy()方法
	time.Sleep(1*time.Second)
	conn.Close()
}

```

- echo server2
```
func server(){
	rx:=regexp.MustCompile("\r\n|\n")

	l,err:=net.Listen("tcp",":2000")
	if err!=nil{log.Fatal(err)}

	for{
		conn,err:=l.Accept()
		if err!=nil{log.Println(err);continue}

		go func(conn net.Conn) {
			io.WriteString(conn,"Enter 'bye' to exit\n")
			r:=bufio.NewReader(conn)

			for{
			    //读取1行再回写
				s,err:=r.ReadString('\n')
				str:=strings.ToLower(rx.ReplaceAllString(s,""))

				if str=="bye"{
					io.WriteString(conn,"Bye\n")
					conn.Close()
					break
				}

				io.WriteString(conn,s)

				if err!=nil{
					conn.Close()
					break
				}
			}
		}(conn)
	}
}

func client(){
	conn,_:=net.Dial("tcp",":2000")

	go func(conn net.Conn) {
		r:=bufio.NewReader(conn)
		for{
			rs,err:=r.ReadString('\n')
			fmt.Print(rs)

			if err!=nil{break}
		}
	}(conn)

	for i:=0;i<5;i++{
		fmt.Fprintln(conn,i)
		time.Sleep(1*time.Second)
	}
	fmt.Fprintln(conn,"bye")

	time.AfterFunc(1*time.Second, func() {
		conn.Close()
	})
}
```