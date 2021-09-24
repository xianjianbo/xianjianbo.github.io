---
title: "Go 语言使用 Slice 指针解决 Slice 扩容的问题"
last_modified_at: 2021-07-16T00:23:17+08:00
---

新手接触 Go 语言 Slice 的时候 len, cap, 扩容等问题常常令人困惑。我们一起来讨论如何解决 Slice 扩容后同步的问题。

## 一些事实
* Go 语言函数参数传递为值传递
* Slice 使用 append 函数增加元素时，如果 len 超过 cap，会进行扩容操作。扩容操作会新开辟一块更大的空间，将新地址返回给 Slice。 

## 例

```go
package main

import "fmt"

func main() {
	sliceA := []int{1, 2, 3, 4, 5}
	fmt.Printf("main before append: len:%d address:%p\n", len(sliceA),sliceA)
	fmt.Println("content:", sliceA)
	changeSliceA(sliceA)
	fmt.Printf("main after append: len:%d address:%p\n", len(sliceA),sliceA)
	fmt.Println("content:", sliceA)
}

func changeSliceA(slicePass []int) {
	fmt.Printf("func before append: len:%d address:%p\n", len(slicePass),slicePass)
	fmt.Println("content:", slicePass)
	slicePass = append(slicePass, 6)
	fmt.Printf("func after append: len:%d address:%p\n", len(slicePass),slicePass)
	fmt.Println("content:", slicePass)
}
```

输出

```
main before append: len:5 address:0xc00009e000
content: [1 2 3 4 5]
func before append: len:5 address:0xc00009e000
content: [1 2 3 4 5]
func after append: len:6 address:0xc0000a6000
content: [1 2 3 4 5 6]
main after append: len:5 address:0xc00009e000
content: [1 2 3 4 5]
```

使用 Slice 指针传值

```go
package main

import "fmt"

func main() {
	sliceA := []int{1, 2, 3, 4, 5}
	fmt.Printf("main before append: len:%d address:%p\n", len(sliceA),sliceA)
	fmt.Println("content:", sliceA)
	changeSliceA(&sliceA)
	fmt.Printf("main after append: len:%d address:%p\n", len(sliceA),sliceA)
	fmt.Println("content:", sliceA)
}

func changeSliceA(slicePass *[]int) {
	fmt.Printf("func before append: len:%d address:%p\n", len(*slicePass),*slicePass)
	fmt.Println("content:", *slicePass)
	*slicePass = append(*slicePass, 6)
	fmt.Printf("func after append: len:%d address:%p\n", len(*slicePass),*slicePass)
	fmt.Println("content:", *slicePass)
}
```

输出

```
main before append: len:5 address:0xc00007a030
content: [1 2 3 4 5]
func before append: len:5 address:0xc00007a030
content: [1 2 3 4 5]
func after append: len:6 address:0xc000014050
content: [1 2 3 4 5 6]
main after append: len:6 address:0xc000014050
content: [1 2 3 4 5 6]
```

