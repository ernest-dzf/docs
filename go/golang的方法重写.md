# golang方法重写

```go
package main

import "fmt"

type IInterface interface {
	Say()
}

type Base struct {
}

func (this Base) Say() {
	fmt.Println("I'm Base")
}

type Derive struct {
	Base
}

func (this Derive) Say() {
	fmt.Println("I'm Derive")
}


func main() {
	var face IInterface

	base := Base{}
	derive := Derive{}
	base.Say()	//I'm Base
	derive.Say() //I'm Derive

	face = derive
	face.Say()	//I'm Derive
	face = base
	face.Say()	//I'm Base
}
```

结构体`Derive`直接内嵌了一个结构体`Base`，这就是golang的继承。

`Base`和`Derive`都实现了`Say()`方法，即实现了`IInterface`接口。