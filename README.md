goexif
======
***Updated at August 25, 2016***

Another example:
```
func parse_file(fname string) {
	f, err := os.Open(fname)
	if err != nil {
		fmt.Println("failed to open file: %s", err.Error())
	}

	x, err := goexif.Decode(f)
	for tag, _ := range x.Main {
		value, err := x.Get(tag)
		if err == nil {
			str := value.String()
			fmt.Printf("%s:%s\n", tag, str)
		} else {
			fmt.Printf("Error parsing tag: %s\n", tag)
		}
	}
}
```

***Updated at July 11th, 2016***

For personal use, export Exif all fields data via `Main` attribute in:

``` go
type Exif struct {
	Tiff *tiff.Tiff
	Main map[FieldName]*tiff.Tag
	Raw  []byte
}
```

So I may walk through all fields like below:

```go
package main

import (
	"log"
	"os"
	"github.com/rwcarlsen/goexif/exif"
)

func ExampleDecode() {
	fname := "sample1.jpg"

	f, err := os.Open(fname)
	if err != nil {
		log.Fatal(err)
	}

	x, err := exif.Decode(f)
	if err != nil {
		log.Fatal(err)
	}

	for key, value := range x.Main {
		log.Println("Field:", key, ", value:", value )
	}
}
```

---

Provides decoding of basic exif and tiff encoded data. Still in alpha - no guarantees.
Suggestions and pull requests are welcome.  Functionality is split into two packages - "exif" and "tiff"
The exif package depends on the tiff package. 
Documentation can be found at http://godoc.org/github.com/rwcarlsen/goexif

Like goexif? - Bitcoin tips welcome: 17M7LFh3ETz4bz83VikB7xuGQskt8K5Lj4

To install, in a terminal type:

```
go get github.com/rwcarlsen/goexif/exif
```

Or if you just want the tiff package:

```
go get github.com/rwcarlsen/goexif/tiff
```

Example usage:

```go
package main

import (
	"fmt"
	"log"
	"os"

	"github.com/rwcarlsen/goexif/exif"
	"github.com/rwcarlsen/goexif/mknote"
)

func ExampleDecode() {
	fname := "sample1.jpg"

	f, err := os.Open(fname)
	if err != nil {
		log.Fatal(err)
	}

	// Optionally register camera makenote data parsing - currently Nikon and
	// Canon are supported.
	exif.RegisterParsers(mknote.All...)

	x, err := exif.Decode(f)
	if err != nil {
		log.Fatal(err)
	}

	camModel, _ := x.Get(exif.Model) // normally, don't ignore errors!
	fmt.Println(camModel.StringVal())

	focal, _ := x.Get(exif.FocalLength)
	numer, denom, _ := focal.Rat2(0) // retrieve first (only) rat. value
	fmt.Printf("%v/%v", numer, denom)

	// Two convenience functions exist for date/time taken and GPS coords:
	tm, _ := x.DateTime()
	fmt.Println("Taken: ", tm)

	lat, long, _ := x.LatLong()
	fmt.Println("lat, long: ", lat, ", ", long)
}
```
