---
layout: post
title: Exploring How Protobuf OneOfs Are Represented
excerpt: "The one where my work doesn't get into production but I do learn something from it."
modified: 2024-12-05T00:00:00+01:00
categories: [misc]
image: https://github.com/user-attachments/assets/0f6d39aa-3af9-4691-8e6c-f17b8a84ac00
tags: [protobug, golang]
comments: true
share: true
---

This is a short exploration of how Protobuf3 OneOf fields are represented using Golang as our exploration medium.

OneOf types, aka [Tagged Unions](https://en.wikipedia.org/wiki/Tagged_union) are data structures that are used to hold one of a finite list of distinct types. A variable of a tagged union type can hold a value of one of several types defined for that tagged union type. This might be easier to understand with an example via pseudocode.

```
type Media = Movie | Show | Short

var favorite1: Media = Movie{}
var favorite2: Media = Show{}
var favorite3: Media = Short{}

var favorites: [Media] = [favorite1, favorite2, favorite3]
```

Variables `favorite1`, `favorite2`, and `favorite3` all have the data type `Media` but have values `Movie`, `Show` and `Short`, which works because the `Media` data type has been defined as a Tagged Union of all 3 other data types. Consequently, the `favorites` variable, which is defined as an array of `Media` values, can have values of the 3 defined data types in it.

Crucially, the values all take up the same space in memory and only one can be set at any given time. My knowledge here becomes fuzzy, but typically, the memory allocated for a OneOf corresponds to the largest possible field within it, with additional space for internal bookkeeping.

[**Protobuf**](https://protobuf.dev/) has [OneOf](https://protobuf.dev/programming-guides/proto3/#oneof) as its implementation of the Tagged Union data type. A message field can be marked as `oneof` with the variants specified.

```protobuf
message Event {
	oneof media {
		Movie movie = 1;
		Show show = 2;
		Short short = 3;
	}
	int64 timestamp = 4;
	optional bool had_fun = 5;
}

message Movie {
	string title = 1;
	string director = 2;
}

message Show {
	string title = 1;
	string runner = 2;
}

message Short {
	string title = 1;
	string shorty = 2; // I couldn't think of something unique, here.
}
```

Protobuf allows assigning only one of the 3 fields out of `movie`, `show` and `short`, and setting one automatically clears any of the others set, ensuring that memory for only one of the types gets consumed.

Protobuf's [`descriptor.proto`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto) is a protobuf file that describes the structure of protobuf files. It hurts my head a little to think about it.

Let's inspect the [`DescriptorProto`](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto#L134) message, which is the schema for a protobuf [Message](https://protobuf.dev/programming-guides/proto3/#simple).
```protobuf
// Describes a message type.
message DescriptorProto {
  optional string name = 1;

  repeated FieldDescriptorProto field = 2;
  ...
  ...
  repeated OneofDescriptorProto oneof_decl = 8;
  ...
}
```

Let's look at the relevant parts of `FieldDescriptorProto` and `OneOfDescriptorProto` and then we'll talk about them.

```protobuf
// Describes a field within a message.
message FieldDescriptorProto {
	...
	// If set, gives the index of a oneof in the containing type's oneof_decl
	// list.  This field is a member of that oneof.
	optional int32 oneof_index = 9;
	...
	// If true, this is a proto3 "optional". When a proto3 field is optional, it
	// tracks presence regardless of field type.
	//
	// When proto3_optional is true, this field must belong to a oneof to signal
	// to old proto3 clients that presence is tracked for this field. This oneof
	// is known as a "synthetic" oneof, and this field must be its sole member
	// (each proto3 optional field gets its own synthetic oneof). Synthetic oneofs
	// exist in the descriptor only, and do not generate any API. Synthetic oneofs
	// must be ordered after all "real" oneofs.
	//
	// For message fields, proto3_optional doesn't create any semantic change,
	// since non-repeated message fields always track presence. However it still
	// indicates the semantic detail of whether the user wrote "optional" or not.
	// This can be useful for round-tripping the .proto file. For consistency we
	// give message fields a synthetic oneof also, even though it is not required
	// to track presence. This is especially important because the parser can't
	// tell if a field is a message or an enum, so it must always create a
	// synthetic oneof.
	//
	// Proto2 optional fields do not set this flag, because they already indicate
	// optional with `LABEL_OPTIONAL`.
	optional bool proto3_optional = 17;
}

// Describes a oneof.
message OneofDescriptorProto {
  optional string name = 1;
  optional OneofOptions options = 2;
}
```

I pasted the whole documentation for the `proto3_optional` field since it has heavy implications on how OneOfs are implemented. `OneOfDescriptorProto` is relatively simple and for us only the `name` field matters.

To fully grasp how to detect and make use of OneOf fields, we must understand both how OneOfs are represented and how optional fields are represented.

> Remember that `optional bool had_fun` field? That wasn't there just for flavor but to demonstrate how optional fields need to be taken into account when making use of OneOfs.

Looking at `DescriptorProto` we see a field `repeated OneofDescriptorProto oneof_decl = 8;`. This field is what lists all OneOf fields in the message. Using our very first example of `message Event`, `oneof_decl` would contain 1 value:

```go
DescriptorProto{
	Name: "Event",
	OneofDecl: []OneofDescriptorProto{
		{
			Name: "media",
		},
		{
			Name: "_had_fun" // Keep an eye on this one. We'll get back to it.
		}
	},
}
```

Wait, what's that `_had_fun` doing in there? We didn't define that as an OneOf! Keep reading.

The fields of the message would have info on each field:

```go
DescriptorProto{
	Name: "Event",
	Field: []FieldDescriptorProto{
		{
			Name: "movie",
			OneOfIndex: 0,
		},
		{
			Name: "show",
			OneOfIndex: 0,
		},
		{
			Name: "short",
			OneOfIndex: 0,
		},
		{
			Name: "timestamp",
			OneOfIndex: nil,
		},
		{
			Name: "had_fun",
			OneOfIndex: 1,
		},
	},
}
```

Of interest to is the value of the `OneOfIndex` property. The description of the property tells us how it works.

> If set, gives the index of an OneOf in the containing type's list. This field is a member of that OneOf.

That seems straightforward enough. In the `Event` message, we have two OneOfs listed (even though we did not list the second one, ourselves). And each of the fields tells us whether it is part of one of the OneOfs (indicating by index which OneOf it belongs to) or not (if the value is `nil`).

As expected, `movie`, `show`, and `short` have the `OneOfIndex` of 0, part of the OneOf located at the 0th index of the `OneofDecl` list, `media`.

But why is `had_fun` is also a part of an OneOf?

Turns out, because of some backwards compatibility shenanigans, optional fields are also represented as OneOfs, with the two options being (I suppose) either the concrete type or nothing.

So marking `had_fun` with the `optional` label causes the descriptor to generate a _synthetic_ OneOf. A OneOf that's not a real OneOf.

When parsing this schema for our own purposes we have to be aware of this synthetic OneOf. We detect _real_ OneOfs by checking if the `OneOfIndex` is not `nil` and the `Proto3Optional` is `false`. If both conditions are met, we can be sure that the field is part of a real OneOf.

All this was discovered over a frustrating afternoon of reading through documentation and experimenting. It was annoying but also rewarding. Hope you enjoyed the read.
