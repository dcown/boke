### 1. unsafe.Pointer
```go
type V struct {
    i int32
    j int64
}

func (this V) PutI() {
    fmt.Printf("i=%d\n", this.i)
}
func (this V) PutJ() {
    fmt.Printf("j=%d\n", this.j)
}

func main() {
    var v *V = new(V)
    fmt.Printf("v: %p\n", v)
    var i *int32 = (*int32)(unsafe.Pointer(v))  // 获取int32地址起始位置
    fmt.Printf("i: %p\n", i)
    *i = int32(981)
    var j *int64 = (*int64)(unsafe.Pointer(uintptr(unsafe.Pointer(v)) + uintptr(unsafe.Sizeof(int64(0)))))
    *j = int64(788)
    fmt.Printf("j: %p\n", j)
    v.PutI()
    v.PutJ()
    
    fmt.Println(unsafe.Sizeof(V{}))
}
// v: 0xc000114000
// i: 0xc000114000
// j: 0xc000114008
// i=981
// j=788
// 16
```
获取j地址时之所以偏移**unsafe.Sizeof(int64(0))**，涉及到内存偏移的问题。
### 2. 获取 slice 长度
```go
// runtime/slice.go
type slice struct {
    array unsafe.Pointer // 元素指针
    len   int // 长度 
    cap   int // 容量
}
func main() {
    s := make([]int, 9, 20)
    Len := *(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(&s)) + uintptr(8)))
    fmt.Println(Len, len(s))
    Cap := *(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(&s)) + uintptr(16)))
    fmt.Println(Cap, cap(s))
}
// 9 9
// 20 20
```

### 3. string 和 slice 的相互转换 实现 zero-copy
只需要共享底层 []byte 数组就可以实现 zero-copy

```go
// reflect
type StringHeader struct {
    Data uintptr
    Len  int
}
type SliceHeader struct {
    Data uintptr
    Len  int
    Cap  int
}
func string2bytes(s string) []byte {
    stringHeader := (*reflect.StringHeader)(unsafe.Pointer(&s))
    bh := reflect.SliceHeader{
        Data: stringHeader.Data,
        Len: stringHeader.Len,
        Cap: stringHeader.Len,
    }
    return *(*[]byte)(unsafe.Pointer(&bh))
}
func bytes2string(b []byte) string {
    sliceHeader := (*reflect.SliceHeader)(unsafe.Pointer(&b))
    s := reflect.StringHeader{
        Data: sliceHeader.Data,
        Len: sliceHeader.Len,
    }
    return *(*string)(unsafe.Pointer(&s))
}

func main() {
    s := "hello"
    fmt.Println(s, "-> ", []byte(s))
    fmt.Println(s, "-> ", string2bytes(s))
    bs := "world"
    b := []byte(bs)
    fmt.Println(b, "->", bs)
    fmt.Println(b, "->", bytes2string(b))
}
```