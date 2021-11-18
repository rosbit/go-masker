# Golang Masker

[![Build Status](https://travis-ci.org/ggwhite/go-masker.svg?branch=master)](https://travis-ci.org/ggwhite/go-masker)
[![codecov](https://codecov.io/gh/ggwhite/go-masker/branch/master/graph/badge.svg)](https://codecov.io/gh/ggwhite/go-masker)
[![Go Report Card](https://goreportcard.com/badge/github.com/ggwhite/go-masker)](https://goreportcard.com/report/github.com/ggwhite/go-masker)
[![License](https://img.shields.io/github/license/mashape/apistatus.svg)](https://github.com/ggwhite/go-masker/blob/master/LICENSE)
[![GoDoc](https://godoc.org/github.com/ggwhite/go-masker?status.svg)](https://godoc.org/github.com/ggwhite/go-masker)
[![Release](https://img.shields.io/github/release/ggwhite/go-masker.svg?style=flat-square)](https://github.com/ggwhite/go-masker/releases/latest)

Golang Masker is a simple utility of creating a mask for sensitive information.

* Changes have been made according to the original version.
* [Getting Started](#Getting-Started)

# Getting Started

```
$ go get -u github.com/rosbit/go-masker
```

## Demo

There are two ways to get a masker instance:
#### 1. Get a instance directly from go-masker package
``` golang
package main

import (
	masker "github.com/rosbit/go-masker"
)

func main() {
	masker.Name("rosbit")
	masker.ID("110101198012310070")
	masker.Mobile("13010010010")
}
```

#### 2. Get a instance via `masker.New()`
``` golang
package main

import (
	masker "github.com/rosbit/go-masker"
)

func main() {
	m := masker.New() // or m := masker.New("*")
	m.Name("rosbit")
	m.ID("110101198012310070")
	m.Mobile("13010010010")
}
```

## Mask Types

|Type        |Const        |Tag        |Description                                                                                            |
|:----------:|:-----------:|:---------:|:------------------------------------------------------------------------------------------------------|
|Name        |MName        |name       |mask the second letter and the third letter                                                            |
|Password    |MPassword    |password   |always return `************`                                                                           |
|Address     |MAddress     |addr       |keep first 6 letters, mask the rest                                                                    |
|Email       |MEmail       |email      |keep domain and the first 3 letters                                                                    |
|Mobile      |MMobile      |mobile     |mask 4 digits from the 4'th digit                                                                      |
|Telephone   |MTelephone   |tel        |remove `(`, `)`, ` `, `-` chart, and mask last 4 digits of telephone number, format to `(??)????-????` |
|ID          |MID          |id         |mask 8 digits from the 6'th digit of ID number                                                         |
|CreditCard  |MCreditCard  |credit     |mask 6 digits from the 7'th digit                                                                      |
|Struct      |MStruct      |struct     |mask the struct                                                                                        |

## Mask the `String`

`String` methomd requires two parameters, a mask type CONST and a string:
``` golang
package main

import (
	masker "github.com/rosbit/go-masker"
)

func main() {
	masker.String(masker.MName, "rosbit")
	masker.String(masker.MID, "110101198012310070")
	masker.String(masker.MMobile, "13010010010")
}
```
Result:
```
r**bit
110101********0070
130****0010
```

## Custom Mask
``` golang
package main

import (
	masker "github.com/rosbit/go-masker"
)

func main() {
	masker.String(masker.MName, "rosbit")
	masker.String(masker.MID, "110101198012310070")
	masker.SetMask("-")
	masker.String(masker.MMobile, "13010010010")
}
```
Result:
```
r**bit
110101********0070
130----0010
```


## Mask the `Struct`

You can define your struct and add tag `mask` to let masker know what kind of the format to mask.

> Field must be **public** in the struct.

``` golang
package main

import (
	"log"
	masker "github.com/rosbit/go-masker"
)

type Foo struct {
	Name   string `mask:"name"`
	Mobile string `mask:"mobile"`
}

func main() {
	foo := &Foo{
		Name:   "rosbit",
		Mobile: "13010010010",
	}
	t, err := masker.Struct(foo)
	log.Println(t)
	log.Println(t.(*Foo))
	log.Println(err)
}
```

Result:
```
t = &{r**bit 130****0010} 
err = <nil>
```

### Struct contain struct

``` golang
package main

import (
	masker "github.com/rosbit/go-masker"
)

type Foo struct {
	Name   string `mask:"name"`
	Mobile string `mask:"mobile"`
	Qoo    *Qoo   `mask:"struct"`
}

type Qoo struct {
	Name      string `mask:"name"`
	Telephone string `mask:"tel"`
}

func main() {
	foo := &Foo{
		Name:   "ggwhite",
		Mobile: "0987987987",
		Qoo: &Qoo{
			Name:      "gino",
			Telephone: "0287658765",
		},
	}
	t, err := masker.Struct(foo)
	log.Println(t)
	log.Println(t.(*Foo).Qoo)
	log.Println(err)
}
```

Result:
```
t = &{g**hite 0987***987 0xc00000a080}
t.Qoo = &{g**o (02)8765-****}
err = <nil>
```

### Struct contain string slice
``` golang
package main

import (
	masker "github.com/rosbit/go-masker"
)

type Foo struct {
	Name   string `mask:"name"`
	Mobile string `mask:"mobile"`
	IDs    []string   `mask:"id"`
}

func main() {
	foo := &Foo{
		Name:   "ggwhite",
		Mobile: "0987987987",
		IDs: []string{
			"A123456789",
			"A987654321",
		},
	}
	t, err := masker.Struct(foo)
	log.Println(t)
	log.Println(err)
}
```

Result:
```
t = &{g**hite 0987***987 [A12345**** A98765****]}
err = <nil>
```
