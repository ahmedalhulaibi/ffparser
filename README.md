# ffparser

The purpose of this package is provide a utility to read a record from a structured [flat-file database](https://en.wikipedia.org/wiki/Flat-file_database) or a record from a text file into a struct. The intent is to eliminate boilerplate code for reading data from a flat file and mapping it to the fields in a struct.

This package allows you to define your record layout mapping using struct tags.

Each field in a struct can be mapped to a single field in a record using a struct tag.

Struct tags are in the form `ffp:"pos,len"` or for a slice field `ffp:"pos,len,occurences"`.

This library provides a method `Unmarshal` which will read a record (slice of bytes) into a struct.

# Usage

Use your favourite dependency tool to pull the code. dep, go mod, etc. Or use go get.

`go get github.com/ahmedalhulaibi/ffparser`

## [Examples](https://github.com/ahmedalhulaibi/ffparser/tree/master/example)

### A simple example of reading a slice of bytes into a struct

```go
package main

import (
	"fmt"

	"github.com/ahmedalhulaibi/ffparser"
)

type CustomerRecord struct {
    //ffparser is one indexed, position starts at 1
	Name        string `ffp:"1,3"`
	OpenDate    string `ffp:"4,10"`
	Age         uint   `ffp:"14,3"`
	Address     string `ffp:"17,15"`
	CountryCode string `ffp:"32,2"`
}

func main() {
	data := []byte("AMY1900-01-01019123 FAKE STREETCA")

	fileHeader := &CustomerRecord{}
	ffparser.Examine(fileHeader)

	err := ffparser.Unmarshal(data, fileHeader, 0)
	fmt.Printf("%v\n", fileHeader)

	if err != nil {
		fmt.Println(err)
	}
}
```


### Example of how to read into struct that contains a slice

```go
package main

import (
	"fmt"

	"github.com/ahmedalhulaibi/ffparser"
)

type CustomerRecord struct {
    //ffparser is one indexed, position starts at 1
	Name        string `ffp:"1,3"`
	OpenDate    string `ffp:"4,10"`
	Age         uint   `ffp:"14,3"`
	Address     string `ffp:"17,15"`
	CountryCode string `ffp:"32,2"`
	//The below tag is in the form "pos,len,occurences"
	//The phone numbers start at position 34 (one indexed)
	//The phone numbers are each number 10 bytes long
	//There are 2 phone numbers total
	//For clarity the second phone number will be read in from pos 44 (one indexed)
	PhoneNumbers []string `ffp:"34,10,2"`
}

func main() {
	data := []byte("AMY1900-01-01019123 FAKE STREETCA41611122229053334444")

	fileHeader := &CustomerRecord{}
	ffparser.Examine(fileHeader)

	err := ffparser.Unmarshal(data, fileHeader, 0)
	fmt.Printf("%v\n", fileHeader)

	if err != nil {
		fmt.Println(err)
	}
}

```



### Example of how to read into a struct that contains an array

```go
package main

import (
	"fmt"

	"github.com/ahmedalhulaibi/ffparser"
)

type CustomerRecord struct {
    //ffparser is one indexed, position starts at 1
	Name        string `ffp:"1,3"`
	OpenDate    string `ffp:"4,10"`
	Age         uint   `ffp:"14,3"`
	Address     string `ffp:"17,15"`
	CountryCode string `ffp:"32,2"`
	//The below tag is in the form "pos,len"
	//The phone numbers start at position 34 (one indexed)
	//The phone numbers are each number 10 bytes long
	//There are 2 phone numbers total
	//For clarity the second phone number will be read in from pos 44 (one indexed)
	PhoneNumbers [2]string `ffp:"34,10"`
}

func main() {
	data := []byte("AMY1900-01-01019123 FAKE STREETCA41611122229053334444")

	fileHeader := &CustomerRecord{}
	ffparser.Examine(fileHeader)

	err := ffparser.Unmarshal(data, fileHeader, 0)
	fmt.Printf("%v\n", fileHeader)

	if err != nil {
		fmt.Println(err)
	}
}

```



# Features

## Data type support:
- [x] bool
- [x] string
- [x] int
- [x] int8
- [x] int16
- [x] int32
- [x] int64
- [x] uint
- [x] uint8
- [x] uint16
- [x] uint32
- [x] uint64
- [x] float32
- [x] float64
- [x] Slice
- [x] Array
- [x] Nested struct

- [x] Slice, Array support AKA Emulate [COBOL occurs clause](https://www.ibm.com/support/knowledgecenter/en/SS6SG3_4.2.0/com.ibm.entcobol.doc_4.2/PGandLR/tasks/tptbl03.htm)

- [x] Offset feature to support reading long lines of data.

    if record exceeds a maximum buffer size, a partial unmarshal can be done
    on the next read, the rest of the data can be unmarshalled into the same struct 
	instance by passing in a position offset

## TODO:
- [ ] Flat File abstraction
- [ ] Support for conditional unmarshal 
    
    if field(pos,len) == "text" do unmarshal else skip. 
    
    This is useful for flat files where there are multiple record layouts within the same file.

- [ ] Byte and Rune support using type override. 

    These are aliases for uint8 and int32 respectively. uint8 and int32 are currenlt parsed as actual numbers not the byte value of the data read in.