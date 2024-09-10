---
layout: post
title: Protopath Problems in Go
excerpt: "The one where I discuss a niche error that troubled me for months."
image: https://github.com/user-attachments/assets/5a8d3279-ada4-4c2c-8714-e8832b4544fe
modified: 2024-09-11T00:00:00+01:00
categories: [misc]
tags: [grpc, golang, protobuf]
comments: true
share: true
---

```shell
Failed to compute set of methods to expose - Symbol not found
```

This cryptic error has reared its ugly head a few times in the past when working with [grpc](https://grpc.io/) [in](https://github.com/golang/protobuf) [Golang](https://go.dev/). I can't remember how I've solved it before. Probably with some combination of stackoverflow answers and random github gists with advice that I have now forgotten. When it happened most recently, I decided to try to actually understand what was going on under the hood.

> Note: For the purposes of this post I am using Golang v1.22.3, [libprotoc](https://github.com/protocolbuffers/protobuf) 3.21.12, [google.golang.org/grpc](https://pkg.go.dev/google.golang.org/grpc) v1.63.2, and [google.golang.org/protobuf](https://pkg.go.dev/google.golang.org/protobuf) v1.34.1.

I had gone a long time without encountering this error, only stumbling upon on it again while using the [protovalidate](https://github.com/bufbuild/protovalidate) library for validating protobuf messages. Used in an [interceptor](https://grpc.io/docs/guides/interceptors/) it provides a way to validate input messages on gRPCs.

According to the [docs](https://github.com/bufbuild/protovalidate?tab=readme-ov-file#usage), I should use the `validate.proto` files by importing them into my project and referencing them in my primary proto files.

```protobuf
syntax = "proto3";

import "buf/validate/validate.proto";

message User {
  // User's name, must be at least 1 character long.
  string name = 1 [(buf.validate.field).string.min_len = 1];
}
```

I am not a fan of having to add files under arbitrary directory structures so I did not store my `validate.proto` (and `expression.proto` and some others that are dependencies of those two) under the `buf` directory. I stored them under the `validate` directory. This is what things looked like.
```sh
├── services.proto
└── validate
    ├── expression.proto
    ├── priv
    │   └── private.proto
    └── validate.proto
```

`services.proto` being my primary proto file of interest. It imported `validate.proto` like so:
```protobuf
import "validate/validate.proto";
```

My generation script does not generate any code from `validate.proto`, since we're using the schema for its options only, that then get baked into the output for my `services.proto` file.

For performing the validations on these baked, generated options, I'm using [github.com/bufbuild/protovalidate-go](https://github.com/bufbuild/protovalidate-go). Code example from its github repo:

```go
package main

import (
	"fmt"
	"time"
	
	pb "github.com/path/to/generated/protos"
	"github.com/bufbuild/protovalidate-go"
	"google.golang.org/protobuf/types/known/timestamppb"
)

func main() {
	msg := &pb.Transaction{
		Id:           1234,
		Price:        "$5.67",
		PurchaseDate: timestamppb.New(time.Now()),
		DeliveryDate: timestamppb.New(time.Now().Add(time.Hour)),
	}

	v, err := protovalidate.New()
	if err != nil {
		fmt.Println("failed to initialize validator:", err)
	}

	if err = v.Validate(msg); err != nil {
		fmt.Println("validation failed:", err)
	} else {
		fmt.Println("validation succeeded")
	}
}
```

Compiling the protobuf into Golang works: the gRPC server registers the RPC server defined in the schema and my program runs. If I call any of the RPCs, it works fine.

It's when I try to use the reflection server is when we run into trouble. I've built a sample project to demonstrate this over at https://github.com/BadgerBadgerBadgerBadger/protoreflect-error-test/tree/main/error-state.

Our `proto` folder looks like this:
```sh
.
├── dependencies
│   └── secondary
│       └── secondary.proto
├── gen.sh
└── primary.proto
```

Our `gen.sh`:
```sh
set -e;  
  
# Get the directory where the script is located  
scriptDir="$( cd -- "$( dirname -- "${BASH_SOURCE[0]:-$0}"; )" &> /dev/null && pwd 2> /dev/null; )";  
echo "script directory: $scriptDir";  

# ensure output directory exists
outDir="$scriptDir/lang_go";  
mkdir -p $outDir/primary;  

# build the secondary proto
protoc -I="$scriptDir/dependencies" \  
    --go_out="$outDir" --go_opt=paths=source_relative \  
    --go-grpc_out="$outDir" --go-grpc_opt=paths=source_relative \  
    secondary/secondary.proto  

# build the primary proto
protoc -I="$scriptDir" \  
    --go_out="$outDir/primary" --go_opt=paths=source_relative \  
    --go-grpc_out="$outDir/primary" --go-grpc_opt=paths=source_relative \  
    primary.proto
```

Our `primary.proto` and `secondary.proto` files:
```protobuf
syntax = "proto3";  
  
option go_package = "github.com/BadgerBadgerBadgerBadger/protoreflect-error-test/proto/lang_go/primary";  
package protoreflect.error.test.primary;  
  
import "dependencies/secondary/secondary.proto";  
  
service Main {  
  rpc GetPrimary (PrimaryRequest) returns (PrimaryResponse) {}  
}  
  
message PrimaryRequest {  
  secondary.SecondaryMessage primary_request = 1;  
}  
  
message PrimaryResponse {  
  secondary.SecondaryMessage primary_response = 1;  
}
```

```protobuf
syntax = "proto3";  
  
option go_package = "github.com/BadgerBadgerBadgerBadger/protoreflect-error-test/proto/lang_go/secondary";  
  
package protoreflect.error.test.secondary;  
  
message SecondaryMessage {  
  string value = 1;  
}
```

I won't paste the go code here that runs the server since it's of less interest to us but you can find that [here](https://github.com/BadgerBadgerBadgerBadger/protoreflect-error-test/blob/main/error-state/cmd/service/main.go).

When we run this, and then try to use [grpcui](https://github.com/fullstorydev/grpcui) to introspect the server, we get:

```sh
➜  ~ grpcui -plaintext localhost:8000
Failed to compute set of methods to expose: Symbol not found: protoreflect.error.test.primary.Main
caused by: File not found: dependencies/secondary/secondary.proto
```

This is the error that gave me a significant amount of headache before I fully understood not only what causes it but also what causes it in my specific situation.

I stumbled across this [github issue](https://github.com/fullstorydev/grpcurl/issues/22). If you're smart (which I am not) you will immediately understand the problem and figure out a solution. It took me a bit longer since I also like to understand how things work.

Since `dependencies/secondary/secondary.proto` is the culprit let's look at two things:
- How are we importing that dependency?
- How are we _building_ that dependency?

Looking at our `primary.proto` file, we are importing it as:

```protobuf
import "dependencies/secondary/secondary.proto";
```

And while the reflection server is trying to find it by that path, it can't seem to.

Looking at our `gen.sh` script, we see:

```sh
protoc -I="$scriptDir/dependencies" \  
    --go_out="$outDir" --go_opt=paths=source_relative \  
    --go-grpc_out="$outDir" --go-grpc_opt=paths=source_relative \  
    secondary/secondary.proto
```

We are _building_ our dependency with the path `secondary/secondary.proto`.

"But Badger", you might ask, "Why does it matter what path we use to reference our dependency while building?" And I might answer: "I didn't think it would, but apparently it does!"

As [jhump](https://github.com/jhump) says in his comment to that github issue I linked:
> this is a problem in the Go protobuf runtime regarding how file descriptors that are compiled into your binary are "linked".
>
> They are linked purely by name. What that means is that the name (and relative path) used to compile a proto with protoc _must exactly match_ how all other files will import it.

Which essentially means that the name you use while building a certain dependency gets encoded in the Golang code of the generated file as the _path_ for that dependency. To me this feels silly, but without jumping into [google.golang.org/protobuf](https://pkg.go.dev/google.golang.org/protobuf)'s code and understanding why they built it this way, it would be similarly silly to pass judgement.

While jhump's example pointed to [this file](https://github.com/jamisonhyatt/grpc-multi-pkg-protos/blob/master/pkg/external/location/location.pb.go#L227) I could not find a similar example in my own code due to the differences in library and tool versions.

I looked at the Golang code generated and followed this code path:
- https://github.com/BadgerBadgerBadgerBadger/protoreflect-error-test/blob/main/error-state/proto/lang_go/secondary/secondary.pb.go#L145
- https://github.com/protocolbuffers/protobuf-go/blob/v1.34.1/internal/filetype/build.go#L121
- https://github.com/protocolbuffers/protobuf-go/blob/v1.34.1/internal/filetype/build.go#L138
- https://github.com/protocolbuffers/protobuf-go/blob/v1.34.1/internal/filedesc/build.go#L91
- https://github.com/protocolbuffers/protobuf-go/blob/v1.34.1/internal/filedesc/build.go#L112
- https://github.com/protocolbuffers/protobuf-go/blob/v1.34.1/reflect/protoregistry/registry.go#L114

I'm sure there's an easier way to do this, but I did it by stepping through the code using [Goland's](https://www.jetbrains.com/go/) debugging tools.

`func (r *Files) RegisterFile(file protoreflect.FileDescriptor) error` is the function responsible for registering a protobuf file to the [Global](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoregistry) [Registry](https://protobuf.dev/reference/go/faq/#namespace-conflict)

Debugging our way to [line 125](https://github.com/protocolbuffers/protobuf-go/blob/v1.34.1/reflect/protoregistry/registry.go#L125) we see that the path has been encoded as `secondary/secondary.proto`, which is the path that we used _when building_ the file.

![[shows-path-of-compiled-protobuf-file.png]]
It looks to be that the path used to build the file is the one that gets embedded as the path _to_ the file. And the gRPC reflection server tries to look up the file using that path.

Since the file does not actually reside at that path, the reflection fails.

The solution is easy enough: use the same path while building the dependency as used to reference the dependency.

The corrected code is at  [https://github.com/BadgerBadgerBadgerBadger/protoreflect-error-test/tree/main/working-state](https://github.com/BadgerBadgerBadgerBadger/protoreflect-error-test/tree/main/working-state)

I hope you enjoyed the post and if you know why things were implemented this way, let me know.
