go-parcel
=========

Encoding and Decoding service for Golang web apps. Out of the box support for JSON, XML, and Querystrings.

```go
package main

import (
	"net/http"

	"github.com/tshaddix/go-parcel"
	"github.com/tshaddix/go-parcel/encoding"
	"github.com/tshaddix/go-parcel/decoding"
)

func main() {
	factory := parcel.NewFactory()

	// Encoders will be called in the
	// order they are registered
	factory.Encoder(encoding.Json())
	factory.Encoder(encoding.Xml())

	// Decoders will be called in the order
	// they are registered
	factory.Decoder(decoding.Query())
	factory.Decoder(decoding.Json())
	factory.Decoder(decoding.Xml())

	myHandler := func(rw http.ResponseWriter, r *http.Request){
		p := factory.Parcel(rw, r)

		// Decode
		err := p.Decode(&someStruct)

		// Decode will now be populated with
		// matching querystrings and json/xml
		// values from request body

		// Do stuff

		// Encode in appropriate response format
		// (xml to xml, json to json)
		err = p.Encode(http.StatusCreated, &someStruct)
	}

	// Build web server
}
```

## BYOD

Bring your own decoder. Here is an example using [gorilla mux](https://github.com/gorilla/mux) params:

```go

import (
	"net/http"

	"github.com/gorilla/mux"
	"github.com/tshaddix/go-parcel"
	"github.com/tshaddix/go-parcel/decoding"
)

type MuxParamStringer struct {}

func MuxParams() *decoding.StringsDecoder {
	return &decoding.StringDecoder{
		new(MuxParamStringer),
		"param",
	}
}

func (self *MuxParamStringer) Len(r *http.Request) int {
	return len(mux.Vars(r))
}

func (self *MuxParamStringer) Get(r *http.Request, name string) string {
	return mux.Vars(r)[name]
}

// Later on

factory.Decoders(MuxParams())

```