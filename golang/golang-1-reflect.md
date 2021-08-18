### 1. reflect.Method
拿到这个接口的函数的定义，然后再对method进行操作

```go
type Person struct {
    Name string
    Age int
    Sex string
}
func (p Person) Say(msg string) (string, error) {
    fmt.Printf("name: %s, age: %d, sex:%s, say hello to %s\n" , p.Name, p.Age, p.Sex, msg)
    return msg, nil
}
func main() {
    p := Person{"lik", 23, "man"}
    typ := reflect.TypeOf(p)  	// 获取结构体类型
    val := reflect.ValueOf(p) 	// 获取结构体值
    fmt.Println("method num: ", typ.NumMethod())  // 方法的个数
    method := typ.Method(0)    	// 获取第一个方法
    f := method.Func
    arg1 := reflect.ValueOf("world")
    out := f.Call([]reflect.Value{val, arg1}) // 调用结构体第一个方法
    fmt.Println(out)
    fmt.Println(out[1].IsNil())
}
// method num:  1
// name: lik, age: 23, sex:man, say hello to world
// [world <error Value>]
// true
```
