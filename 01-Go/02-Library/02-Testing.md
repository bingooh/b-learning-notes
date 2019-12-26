## Misc
- 签名规范
    - `TestXxx(*testing.T)` 测试方法
    - `xxx_test.go`         测试源文件
- `go test` 执行测试，默认执行当前包下的所有测试方法
    - `go test -v -cover`   显示测试详细信息 
    - `go test -run ''`     执行所有单元测试
    - `go test -run Foo`    执行匹配`Foo`的主测试，如`TestFooBar`
    - `go test -run Foo/A=` 执行匹配`Foo`的主测试下的所有匹配`A=`的子测试
    - `go test -run /A=1`   执行所有主测试下所有匹配`A=1`的子测试
- `*testing.T`包含以下方法用于报告测试结果
    - `Fail()`           报告测试失败，但继续执行当前测试
    - `FailNow()`        报告测试失败并退出当前测试，继续执行下1测试
    - `Log()/Logf()`     输出日志，仅在测试失败或使用`go test -v`时输出
    - `Error()/Errorf()` 顺序调用`Log()->Fail()`
    - `Fatal()/Fatalf()` 顺序调用`Log()->FailNow()`
- `t.Parallel()` 并行执行多个测试，默认串行执行
- `t.Run()` 执行子测试
- 参考文档
    - [`go test`命令参数](https://golang.org/cmd/go/#hdr-Testing_flags)
```
//被测试方法
func Hi(v string) string {
	return "hi," + v
}

//table test
func TestHi(t *testing.T) {
	cases := []struct{ in, expect string }{
		{"1", "hi,1"},
		{"2", "hi,2"},
	}

	for _, c := range cases {
		result := Hi(c.in)
		if result != c.expect {
			t.Errorf("Hi(%q)==%q, expect %q", c.in, result, c.expect)
		}
	}
}

//跳过测试，测试命令：go test -v -short
func TestLong(t *testing.T) {
	if testing.Short(){
		t.Skip("skip TestLong")
	}

	time.Sleep(3*time.Second)
	t.Log("TestLong done");
}

//以下方法在main协程执行，并将在所有其他测试开始前/后执行
func TestMain(m *testing.M) {
	// call flag.Parse() here if TestMain uses flags
	fmt.Println("testing start")
	code:=m.Run()
	fmt.Println("testing done")

	os.Exit(code)
}
```

## Benchmark
- `BenchmarkXxx(*testing.B)` 基准测试方法签名
- `go test -run "none" -bench regexp` 仅执行基准测试
- 基准测试方法会执行足够多的次数，直到获取可靠的测试结果
```
//测试命令：go test -run "none" -bench "Hi"
//如果初始化很耗时，可先b.StopTimer(),初始化完成后再b.StartTimer()
func BenchmarkHi(b *testing.B) {
	b.ResetTimer() //重置计时器
	for i := 0; i < b.N; i++ {
		fmt.Sprintln(Hi(strconv.Itoa(rand.Int())))
	}
}
```

## Example
- `Example`方法将会作为示例写入帮助文档
- `Example`方法命名规范
    - `F(函数)`,`T(类型)`,`M(方法)`,
    - `_suffix(同1目标的多个示例的后缀)`
    - `Example()`    包示例
    - `ExampleF()`   函数示例
    - `ExampleT()`   类型示例，如接口示例
    - `ExampleT_M()` 方法示例，如接口方法示例
- `// Output:` 指定期望的输出到`Stdout`的数据，以便测试示例方法
```
//测试命令：go test -run "Example"
func ExampleHi(){
	result:=Hi("world")

	fmt.Println(result)
	//Output:
	//hi,world
}

//Hi()的第2个示例
func ExampleHi_2(){
	result:=Hi("bingo")

	fmt.Println(result)
	//Output:
	//hi,bingo
}
```