---
title: "Intro to protobuf (with ruby)"
date: 2023-01-30T22:49:00+02:00
draft: false
categories:
  - Learning
---

Protobuf or more precisely Protocol Buffers is a data serialization format developer by Google and it is designed to be language-agnostic. What's important (and what makes it a better choice than ie. JSON) is that it automatically provides validation of data, preserves order in arrays and provides pre-generated classes that do all the hard work with set of setters and getters. Magic is done by compiling `.proto` files with schema into language files.

When I was trying to learn more about it (to actually learn what it is and why I'd like to use it), I've noticed that in [examples directory](https://github.com/protocolbuffers/protobuf/tree/main/examples) of protobuf project on Github the ruby example is missing, even thoughruby support is build-in into `protoc` compiler. So I've decided to fix that and a ruby example now can be found in [my fork](https://github.com/mlitwiniuk/protobuf/tree/ruby_example/examples)

Lessons learned here:

1. array objects are added via `push` method, ie:

```ruby
address_book.people.push(person)
```

2. enum comparison is done via symbols:

```ruby
phone_number.type = :MOBILE
# and not through constants, as those return enum index:
Tutorial::Person::PhoneNumber::MOBILE == 0
```

3. Package from `.proto` files becomes initial namespace, ie `package tutorial` will result in objects embedded in `Tutorial` module:

```ruby
module Tutorial
  Person = ::Google::Protobuf::DescriptorPool.generated_pool.lookup("tutorial.Person").msgclass
  Person::PhoneNumber = ::Google::Protobuf::DescriptorPool.generated_pool.lookup("tutorial.Person.PhoneNumber").msgclass
  Person::PhoneType = ::Google::Protobuf::DescriptorPool.generated_pool.lookup("tutorial.Person.PhoneType").enummodule
  AddressBook = ::Google::Protobuf::DescriptorPool.generated_pool.lookup("tutorial.AddressBook").msgclass
end

```

4. to create protobuf mapping from source `.proto` file you run:

```bash
$ protoc --ruby_out=. addressbook.proto
# will result in new addressbook_pb.rb file
```

I still need to check out [Avro](https://avro.apache.org), but protobuf seems to be perfect data exchange format between services / applications in different languages and developed by different teams.
