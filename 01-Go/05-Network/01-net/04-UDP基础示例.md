## 示例
- 有连接UDP
```
func server(){
	conn,err:=net.ListenPacket("udp4",":8000")
	if printErr(err,"server conn err"){os.Exit(1)}

	fmt.Printf("client connected: %s\n",conn.LocalAddr())

	bs:=make([]byte,512)
	for{
		n,addr,err:=conn.ReadFrom(bs)  //每次读取1个packet
		printErr(err,"server read err")

        //addr即客户端的接收地址
		fmt.Printf("server received packet from: %s\n",addr)

        //发送数据，不能使用conn.(*net.UDPConn).Write()
		_,err=conn.WriteTo(bs[:n],addr)
		printErr(err,"server write err")
	}
}

func client(msg string){
	conn,err:=net.Dial("udp4",":8000")
	if printErr(err,"conn err"){os.Exit(1)}

	go func(conn net.Conn) {
		io.Copy(os.Stdout,conn)
	}(conn)

    //发送数据，可以使用conn.(*net.UDPConn).Write()
	time.Sleep(1*time.Second)
	fmt.Fprintln(conn,msg)

	time.Sleep(1*time.Second)
	conn.Close()
}

func main() {
	go client("Hello")
	go client("World")
	server()
}
```

- 无连接UDP
```
// 监听源地址，发送数据给目标地址
func peer(src,dst string){
	conn,_:=net.ListenPacket("udp4",src)

	go func(conn net.PacketConn) {
		b:=make([]byte,512)
		n,_,_:=conn.ReadFrom(b)
		fmt.Printf("%s recv msg: %s",src,b[:n])
	}(conn)

	msg:=fmt.Sprintf("%s say hi!\n",src)
	dstAddr,_:=net.ResolveUDPAddr("udp4",dst)
    
    //无需等待客户端连接
	conn.WriteTo([]byte(msg),dstAddr)
}

func main() {
	addr1,addr2:="127.0.0.1:8000","127.0.0.1:8001"
	go peer(addr1,addr2)
	go peer(addr2,addr1)

	time.Sleep(10*time.Second)
}
```