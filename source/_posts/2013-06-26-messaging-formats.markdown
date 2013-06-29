---
layout: post
title: "Messaging Formats"
slug: "messaging-formats"
date: 2013-06-26 15:42
comments: true
categories:
- c++
- json
- xml
- msgpack
- protobuf
tags:
- c++
- json
- xml
- msgpack
- protobuf
---

With ever increasing data volumes being transfered in networks around the world, the type data formats used to pass information around has obviously been a heavily discussed topic. It ussually comes down to three main factors: data compression, speed of encoding/decoding and how easy it is to read and debug. In this post I am going to talk about some of the most commonly used message formats; their advantages and dissadvantages, including my opinions, and links to some other blogs posts with examples of using them with c++.

<!-- more -->

Text Formats
------------

Lets first of all split these messaging formats up into two subcategories. First of all we have human readible format. These have the advantage that it's very easy to debug and understand what is being sent over the wire, although obviously the size of data is quite a bit larger than it needs to be. The obvious contenders here are Json, XML and YAML. Json seems to be the most used as it is very dynamic, simple and works really well with javascript where a lot of restfull web services are used. XML used the be the most popular and is still very popular, but it is slightly more complicated to use than json. I'll only discuss json and xml below as I think these are the most widely used text based messaging formats and there are quite a few competing c++ libraries for each.

### Json

[Json C++ Examples](/blog/2013/06/27/json-c-plus-plus-examples/)

Json is simply made up of a combination of of arrays and objects (aka dictionaries or maps depending on you programming affiliation), using javascript syntax, where the object keys are strings, and the items in the array and objects values can be arrays, objects, strings, numbers, booleans or null. This is an example of the json format:

{% codeblock example.json lang:json %}

{
  "hello": "world",
  "array": [
    "hello",
    "world"
  ],
  "number": 0,
  "object": {
      "hello": "world"
  }
}

{% endcodeblock %}

### Xml

[Xml C++ Examples](/blog/2013/06/28/xml-c-plus-plus-examples/)

Xml is made up of a hierarchy of tags and values. Each tag may also have its own properties. Xml is a very mature message structure and as such it has a few very well defined schema defination languages that can be used to make sure xml files conform to a certain structure, with the most popular being the Xml Schema Definition (xsd). An example of an xsd is below, and it will be used in one of our examples that uses the xsd to generate code to encode and decode to and from xml.

{% codeblock address.xsd lang:xml %}

<?xml version="1.0" encoding="utf-8"?>
<xs:schema elementFormDefault="qualified" xmlns:xs="http://www.w3.org/2001/XMLSchema">
  <xs:element name="Address">
    <xs:complexType>
      <xs:sequence>
        <xs:element name="Recipient" type="xs:string" />
        <xs:element name="House" type="xs:string" />
        <xs:element name="Street" type="xs:string" />
        <xs:element name="Town" type="xs:string" />
        <xs:element name="County" type="xs:string" minOccurs="0" />
        <xs:element name="PostCode" type="xs:string" />
        <xs:element name="Country" minOccurs="0">
          <xs:simpleType>
            <xs:restriction base="xs:string">
              <xs:enumeration value="IN" />
              <xs:enumeration value="DE" />
              <xs:enumeration value="ES" />
              <xs:enumeration value="UK" />
              <xs:enumeration value="US" />
            </xs:restriction>
          </xs:simpleType>
        </xs:element>
      </xs:sequence>
    </xs:complexType>
  </xs:element>
</xs:schema>

{% endcodeblock %}

With an example of an xml message conforming to the above schema being.

{% codeblock address.xml lang:xml %}

<?xml version="1.0" encoding="utf-8"?>
<Address xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:noNamespaceSchemaLocation="address.xsd">
  <Recipient>Mr. Walter C. Brown</Recipient>
  <House>49</House>
  <Street>Featherstone Street</Street>
  <Town>LONDON</Town>
  <PostCode>EC1Y 8SY</PostCode>
  <Country>UK</Country>
</Address>

{% endcodeblock %}

Binary Formats
--------------

[Binary Message Format C++ Examples](/blog/2013/06/29/binary-message-format-c-plus-plus-examples/)

Then we have non human-readilble messaging. These use some sort of binary representation when going across the wire but need to be decoded and encoded either side. These have been given the name of "protocol buffers". The two mainly used ones are Google's protobuf and msgpack. The advantages of these are that the payloads of data you need to send are a lot smaller compared with a human readible format. But then you lose the ability to understand (without computer interaction) what is being sent across the wire. You are also going to need a bit more time at each end to encode and decode, but protocol buffers are heavily optimised and should not take too long to encode/decode. Both of these formats have many bindings in multiple programming languages to allow you to communicate between any system you need. There seems to be a lot of debate currently about which is faster out of the two, and I suggest you profile the two for your specific needs and actually decide which to use taking into account your own profiling and the fundamental differences between the two libraries.

Protobuf
--------

Protobuf requires a schema to keep data consistent and at least in c++ uses this schema to generate optimised code. This gives you methods, depending on the datatypes defined in the schema, to access and set any value you defined. You can not add any value into a protobuf message that is not defined in the schema nor one that has an incorrect type. An example of a schema taken from the protobuf documentation is:

{% codeblock protobuf proto file lang:cpp %}

message Person {
  required string name = 1;
  required int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    required string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phone = 4;
}

message AddressBook {
  repeated Person person = 1;
}

{% endcodeblock %}

This defines an address book which stores a list of people and their phone numbers, specifying what ntype the number is.

### Protobuf Binary Format

#### Varints

Protobuf heavily uses varints which are a way to represent numbers with a varying amount of memory. In the case of protobufs it does this by setting the the first bit to 1 in every byte if there are further bytes to come. The other 7 bits of each byte are used to store the two's complement representation of the number in groups of 7 bits using the **least significant group first** i.e. you need to reverse the groups of 7 bits. Below is an example of working from the binary representation of a varint to a negative decimal number:

    1011 1000 0011 1100

First take off the first bit from each byte.

    011 1000 011 1100

Then reverse the byte order as the protocol defines that the least significant group comes first.

    111 1100 111 0000

Then finally convert the two's compliment binary number into a decimal.

    -400

#### Message Structure

Like with most messaging format a protocol buffer message is a series of key-value pairs. In the binary format the key is made up from the fields number from the .proto file and a number defining the field type. The use of the field number from the schema saves quite a lot of space, however, means that at each end there is a requirement for a .proto file referencing that field. The field type is needed so that protobuf can work out the length of the value so as to skip over any it does not recognise, i.e. to allow backwards compatability of .proto files.

Each key in the binary message is a varint with the value:

    (field_number << 3) | type

i.e. the last three bits of the number specify the type, these values are hardcoded and defined in the protobuf documentation.

The possible value types are pretty much just a combination of varints, fixed-length values and length-delimited values. With length-delimited values (string, bytes, etc...) the following field is just a varint defining the length in bytes of the value and then the actual value.

Msgpack
-------

Msgpack is very dynamic and bases itself on json except that at either end it encodes/decodes into a binary protocol. Even when you output it out inside the code it prints it out as if it were encoded in json. Since it allows any data to be added and removed it generally uses the language own containers (lists and maps) as the undecoded format. The problem with the use of this library with static languages is that it requires hacks and quite a lot of boilerplate code to get around the static limititation that appear when message structure becomes more complicated.

Msgpack Binary Format
---------------------

Msgpack contains all type information inside the binary message and therefore is always backwards compatible. Each value is stored in a **type-data** or **type-length-data** style. Meaning that there are quite a few well defined types that have a hardcoded fixed length value and others such as raw bytes and containers need to know the lenght of their values.

Summary
-------

I think with the modern day responsive websites mostly utilising restfull web api's that json is here to stay and xml will slowly fade away. Even xml configuration files (a very popular use of xml) are slowly moving to json equivalents.

I see no real reason to replace restfull json api's with binary protocl buffers considering with json it is very quick to convert into javascript objects and since with gzipping api respones the size of json can be greatly reduced. However, in other multi-platform architectures, especially real-time systems or systems dealing with very large data volumes, I believe that there is tremendous value in using a binary representation across the wire.

Which binary representation to use depends heavily on the particular use case. But as my examples in c++ show. I strongly feel that using msgpack in a statically typed language greatly reduces the impressiveness of msgpack's dynamic features.
