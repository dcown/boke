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
### 4. reflect.Indirect
func Indirect(v Value) value    
Indirect returns the value that v points to. If v is a nil pointer, Indirect returns a zero Value. If v is not a pointer, Indirect returns v.

```go
func main() {
    var str []string
    strValue := reflect.ValueOf(&str)
    indirectValue := reflect.Indirect(strValue)  // 获取nil指针的Value
    valueSlice := reflect.MakeSlice(indirectValue.Type(), 100, 1024)
    kind:= valueSlice.Kind()
    cap:= valueSlice.Cap()
    length:= valueSlice.Len()
    fmt.Printf("Type is [%v] with capacity of %v bytes"+ " and length of %v .\n", kind, cap, length)
}
// Type is [slice] with capacity of 1024 bytes and length of 100 .
```
### 5. reflect.Type.Implement
检测是否实现了某个接口规范

```go
type Model interface {
    m()
}
func HasModels(m Model) {
    s := reflect.ValueOf(m).Elem()
    t := s.Type()
    modelType := reflect.TypeOf((*Model)(nil)).Elem()
    for i := 0; i < s.NumField(); i++ {
        f := t.Field(i)
        fmt.Printf("%d: %s %s -> %t\n", i, f.Name, f.Type, f.Type.Implements(modelType))
    }
}

type Company struct{}

func (Company) m() {}

type Department struct{}

func (*Department) m() {}

type User struct {
    CompanyA    Company
    CompanyB    *Company
    DepartmentA Department
    DepartmentB *Department
}

func (User) m() {}

func main() {
    HasModels(&User{})
}
// 0: CompanyA main.Company -> true
// 1: CompanyB *main.Company -> true
// 2: DepartmentA main.Department -> false
// 3: DepartmentB *main.Department -> true
``` 
### 6. reflect.AssignableTo
```go
type User struct {
    Name string
    Age  int64
    Sex  string
}

func main() {
    user := User{"张三", 25, "男"}
    FillStruct(user)
}

func FillStruct(obj interface{}) {
    t := reflect.TypeOf(obj)       //反射出一个interface{}的类型
    fmt.Println(t.PkgPath())       //反射对象所在的短包名
    fmt.Println(t.String())        //包名.类型名
    fmt.Println(t.Size())          //要保存一个该类型要多少个字节
    fmt.Println(t.Align())         //返回当从内存中申请一个该类型值时，会对齐的字节数
    fmt.Println(t.FieldAlign())    //返回当该类型作为结构体的字段时，会对齐的字节数

    var u User
    fmt.Println(t.AssignableTo(reflect.TypeOf(u)))  // 如果该类型的值可以直接赋值给u代表的类型，返回真
    fmt.Println(t.ConvertibleTo(reflect.TypeOf(u))) // 如该类型的值可以转换为u代表的类型，返回真

    fmt.Println(t.NumField())             // 返回struct类型的字段数（匿名字段算作一个字段），如非结构体类型将panic
    fmt.Println(t.Field(0).Name)          // 返回struct类型的第i个字段的类型，如非结构体或者i不在[0, NumField())内将会panic
    fmt.Println(t.FieldByName("Age"))     // 返回该类型名为name的字段（会查找匿名字段及其子字段），布尔值说明是否找到，如非结构体将panic
    fmt.Println(t.FieldByIndex([]int{2})) // 返回索引序列指定的嵌套字段的类型，等价于用索引中每个值链式调用本方法，如非结构体将会panic
}
// main
// main.User
// 40
// 8
// 8
// true
// true
// 3
// Name
// {Age  int64  16 [1] false} true
// {Sex  string  24 [2] false}
```