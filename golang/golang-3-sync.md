### 1. [sync.Pool](https://zhuanlan.zhihu.com/p/133638023)
对象的复用，避免重复创建、销毁

```go
var pool *sync.Pool

type Person struct {
    Name string
}

func initPool() {
    pool = &sync.Pool {
        New: func()interface{} {
            fmt.Println("Creating a new Person")
            return new(Person)
        },
    }
}

func main() {
    initPool()

    p := pool.Get().(*Person)
    fmt.Println("首次从 pool 里获取：", p)

    p.Name = "first"
    fmt.Printf("设置 p.Name = %s\n", p.Name)

    pool.Put(p)

    fmt.Println("Pool 里已有一个对象：&{first}，调用 Get: ", pool.Get().(*Person))
    fmt.Println("Pool 没有对象了，调用 Get: ", pool.Get().(*Person))
}
// Creating a new Person
// 首次从 pool 里获取： &{}
// 设置 p.Name = first
// Pool 里已有一个对象：&{first}，调用 Get:  &{first}
// Creating a new Person
// Pool 没有对象了，调用 Get:  &{}
```