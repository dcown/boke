### 1. [reflect.Type 和 reflect.Value](https://www.jianshu.com/p/26a284e69586)
interface{}类型变量具体类型可以使用reflect.Type来表示，其具体值则使用reflect.Value来表示
```go
type order struct {
    ordId      int
    customerId int
}

type employee struct {
    name    string
    id      int
    address string
    salary  int
    country string
}

func query(q interface{}) {
    val := reflect.ValueOf(q)
    if val.Kind() == reflect.Struct {
        name := reflect.TypeOf(q).Name()
        sql := fmt.Sprintf("insert into %s values(", name)
        for i := 0; i < val.NumField(); i++ {
            switch val.Field(i).Kind(){
            case reflect.Int:
                if i == 0 {
                    sql = fmt.Sprintf("%s%d", sql, val.Field(i).Int())
                } else {
                    sql = fmt.Sprintf("%s, %d", sql, val.Field(i).Int())
                }
            case reflect.String:
                if i == 0 {
                    sql = fmt.Sprintf("%s%s", sql, val.Field(i).String())
                } else {
                    sql = fmt.Sprintf("%s, %s", sql, val.Field(i).String())
                }
            default:
                fmt.Println("not support")
                return
            }
        }
        sql = fmt.Sprintf("%s)", sql)
        fmt.Println(name, " sql: ", sql)
        return

    }
    fmt.Println("unsupported type")
}

func main() {
    o := order{
        ordId:      456,
        customerId: 56,
    }
    query(o)

    e := employee{
        name:    "Naveen",
        id:      565,
        address: "Coimbatore",
        salary:  90000,
        country: "India",
    }
    query(e)

    i := 90  // 这是一个错误的尝试
    query(i)
}
// order  sql:  insert into order values(456, 56)
// employee  sql:  insert into employee values(Naveen, 565, Coimbatore, 90000, India)
// unsupported type
```
### 2. reflect.MakeSlice, reflect.MakeMap
```go
func main() {
    intSlice := make([]int, 0)
    mapStringInt := make(map[string]int)

    sliceTyp := reflect.TypeOf(intSlice)
    mapTyp := reflect.TypeOf(mapStringInt)

    intSliceReflect := reflect.MakeSlice(sliceTyp, 0, 0)
    mapReflect := reflect.MakeMap(mapTyp)

    v := 10
    rv := reflect.ValueOf(v)
    intSliceReflect = reflect.Append(intSliceReflect, rv)
    s2 := intSliceReflect.Interface().([]int)
    fmt.Printf("%v, %T\n", intSliceReflect, intSliceReflect)
    fmt.Printf("%v, %T\n", s2, s2)

    k := "name"
    rk := reflect.ValueOf(k)
    mapReflect.SetMapIndex(rk, rv)
    m2 := mapReflect.Interface().(map[string]int)
    fmt.Printf("%#v, %T\n", mapReflect, mapReflect)
    fmt.Printf("%#v, %T\n", m2, m2)
}
// [10], reflect.Value
// [10], []int
// map[string]int{"name":10}, reflect.Value
// map[string]int{"name":10}, map[string]int
```

### 3. reflect.Method
拿到这个接口的函数的定义，然后再对method进行操作.  
Methods are sorted in lexicographic order.以字典的顺序进行方法存储

```go
type Person struct {
    Name string
    Age int
    Sex string
}

func (p Person) Add(msg string, x ... int) {
    fmt.Println("add val msg: ", msg)
    fmt.Println("add val x:", x)
}

func (p Person) Say(msg string) (string, error) {
    fmt.Printf("name: %s, age: %d, sex:%s, say hello to %s\n" , p.Name, p.Age, p.Sex, msg)
    return msg, nil
}

func main() {
    p := Person{"lik", 23, "man"}
    typ := reflect.TypeOf(p)    // 获取结构体类型
    val := reflect.ValueOf(p)   // 获取结构体值
    fmt.Println("method num: ", typ.NumMethod())
    say := typ.Method(1)
    f := say.Func
    arg1 := reflect.ValueOf("world")
    out := f.Call([]reflect.Value{val, arg1})
    fmt.Println(out)
    fmt.Println(out[1].IsNil())

    add := typ.Method(0).Func
    // in2 := add.Type().In(2)     // 第二个参数的类型
    intSlice := make([]int, 0)
    in2 := reflect.TypeOf(intSlice)

    argslice := []int {1, 2, 5}
    x := reflect.MakeSlice(in2, 0, 0)  // 初始化第二个参数
    for i := 0; i < len(argslice); i++ {
         x = reflect.Append(x, reflect.ValueOf(argslice[i]))
    }
    in := []reflect.Value{val, arg1}
    in = append(in, x)
    add.CallSlice(in)
}
// method num:  2
// name: lik, age: 23, sex:man, say hello to world
// [world <error Value>]
// true
// add val msg:  world
// add val x: [1 2 5]
```
