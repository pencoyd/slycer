# slycer

A library for easily moving around in Go byte slices. The goal is for it to be
used as both a diagnostic tool for mapping out binary strucutres, and clean
index management for tracking and stepping through a slice.

## Example

```go
package main

import (
	"fmt"

	"github.com/invisiblethreat/slycer"
)

func main() {

	b := []byte{
		0x00, 0x00, 0x00, 0x31, 0xff, 0x53, 0x4d, 0x42, 0x72, 0x00, 0x00, 0x00, 0x00, 0x18, 0x45, 0x68,
		0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x00, 0x2f, 0x09,
		0x00, 0x00, 0x01, 0x00, 0x00, 0x0e, 0x00, 0x02, 0x4e, 0x54, 0x20, 0x4c, 0x4d, 0x20, 0x30, 0x2e,
		0x31, 0x32, 0x00, 0x02, 0x00}

	null := byte(0x00)

	o := slycer.NewOffsetTracker()
	o.SetMax(len(b))
	o.SkipNote(3, "header")
	length := byteToInt(b[o.Index():o.Step(1)])
	fmt.Printf("Length: %d\n", length)
	o.SaveCurrent("slide")
	for {
		if b[o.Index()] == null && !o.ExceedsMax(o.Index()) {
			break
		} else {
			o.Step(1)
		}
	}
	notNull := b[o.GetSaved("slide"):o.Index()]
	fmt.Printf("Not null bytes: %#v\n", notNull)
	o.SkipNote(4, "reserved")
	o.ShowSaved()
	o.ShowSkipped()

}

func byteToInt(b []byte) int {
	if len(b) > 1 {
		panic("and freak out.")
	}
	return int(b[0])
}
```



```shell
go run example.go
Length: 49
Not null bytes: []byte{0xff, 0x53, 0x4d, 0x42, 0x72}
const (
        slide = 4
)
const (
        header = 0
        reserved = 9
)
```