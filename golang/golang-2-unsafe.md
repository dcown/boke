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