---
layout: post
title: "Binary Message Format C++ Examples"
slug: "binary-message-format-c++-examples"
date: 2013-06-29 15:44
comments: true
categories:
- c++
- msgpack
- protobuf
tags:
- c++
- msgpack
- protobuf
---

Protobuf C++ Example
--------------------

Below is the schema for an address book example, this is used to generate c++ code using the protobuf compiler, using this command to put the generated code into the "generated" directory: protoc AddressBook.proto --cpp_out=generated. As you would expect it generates two files, a header and an implementation file. These need to be included and compiled, respectively, with the code using them.

<!-- more -->

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

Below the protobuf is created and in this case is serialized to a string value, there are many protobuf methods to serialize the protobuf into different formats but with string you do not have to worry about memory allocation.

{% codeblock create protobuf lang:cpp %}

void createProtobuf(std::string & data)
{
    AddressBook address_book;

    // Add an address.
    Person* person = address_book.add_person();

    person->set_id(1);
    person->set_name("Thomas Whitton");
    person->set_email("mail@thomaswhitton.com");

    Person::PhoneNumber* phone_number = person->add_phone();
    phone_number->set_number("123456890");

    phone_number->set_type(Person::MOBILE);

    address_book.SerializeToString(&data);
}

{% endcodeblock %}

Not too much interesting code below, all it does is output the protobuf message defined above. Useful for understanding the different getters available in the protobuf api.

{% codeblock output protobuf lang:cpp %}

void outputAddresBook(const AddressBook &address_book)
{
    for (int i = 0; i < address_book.person_size(); ++i)
    {
        const Person& person = address_book.person(i);

        std::cout << "Person ID: " << person.id() << std::endl;
        std::cout << "  Name: " << person.name() << std::endl;
        if (person.has_email())
        {
            std::cout << "  E-mail address: " << person.email() << std::endl;
        }

        for (int j = 0; j < person.phone_size(); ++j)
        {
            const Person::PhoneNumber& phone_number = person.phone(j);

            switch (phone_number.type())
            {
                case Person::MOBILE:
                    std::cout << "  Mobile phone #: ";
                    break;
                case Person::HOME:
                    std::cout << "  Home phone #: ";
                    break;
                case Person::WORK:
                    std::cout << "  Work phone #: ";
                    break;
            }
            std::cout << phone_number.number() << std::endl;
        }
    }
}

{% endcodeblock %}

Below is the code that brings the other two snippets together, first we check for protobuf version using a special protobuf macro, then we store the protobuf binary into a string, then we parse this binary data into a completely new protobuf message, then we output the new message using the method defined above. Also it is good practice to call the ShutDownProtobufLibrary method once you have finished using protobuf code. Most applications probably will not have to do this since the program will terminate anyway once it has stopped using protobuf.

{% codeblock protobuf.cpp lang:cpp %}

// Verify that the version of the library that we linked against is
// compatible with the version of the headers we compiled against.
GOOGLE_PROTOBUF_VERIFY_VERSION;

std::string data;

createProtobuf(data);

AddressBook address_book;

address_book.ParseFromString(data);

outputAddresBook(address_book);

// Optional:  Delete all global objects allocated by libprotobuf.
google::protobuf::ShutdownProtobufLibrary();

{% endcodeblock %}

Msgpack C++ Example
-------------------

In c++ if the messages created from vectors, maps and PODs then msgpack will automatically covert data back into typed objects. Unfortunately for use with c++ if you require the message format as an array or map that contains different types then one must use helper functions to allow serialising of different types in the same message one after another (order will matter). Or you can create a class or struct in c++ that can hold these values, and then call the macro MSGPACK\_DEFINE to tell msgpack what variables are going to be part of the serialisation, this can also be used to serialise custom classes.

The simple example of msgpack i.e. when you serialise just vectors, maps and PODs it can be performed very easily as can be seen below:

{% codeblock simple msgpack lang:cpp %}

// ---- create the structure that will be serialised ----
std::vector<std::string> vec;
vec.push_back("Hello");
vec.push_back("World");

// ---- serialize it into buffer ----
msgpack::sbuffer sbuf;
msgpack::pack(sbuf, vec);

// ---- deserialize it ----
msgpack::unpacked msg;
msgpack::unpack(&msg, sbuf.data(), sbuf.size());

msgpack::object obj = msg.get();

// ---- convert msgpack objects into statically typed object ----
std::vector<std::string> rvec;
obj.convert(&rvec);

std::cout << vec[0] << " " << vec[1] << std::endl;

{% endcodeblock %}

Msgpack provides a packer object that can pack multiple types into a single msgpack message. This requires a little more code, but it is still pretty easy to follow. Then it's pretty simple code to unpack the message using the unpacker object.

{% codeblock msgpack packing lang:cpp %}

// ---- create the structure that will be serialised ----
std::vector<std::string> vec;
vec.push_back("Hello");
vec.push_back("World");

std::map<std::string, std::string> map;
map["hello"] = "world";

// ---- serialize it into buffer ----
msgpack::sbuffer buffer;
msgpack::packer<msgpack::sbuffer> pk(&buffer);
pk.pack(std::string("hello"));
pk.pack(vec);
pk.pack(map);

// ---- de-serialise these objects using msgpack::unpacker ----
msgpack::unpacker pac;

// copy the buffer data to the unpacker object
pac.reserve_buffer(buffer.size());
memcpy(pac.buffer(), buffer.data(), buffer.size());
pac.buffer_consumed(buffer.size());

// deserialize it.
msgpack::unpacked msg;

std::string outputString;
std::vector<std::string> outputVector;
std::map<std::string, std::string> outputMap;

// ---- convert msgpack objects into statically typed objects ----

// unpsck objects in the order in which they were packed
pac.next(&msg);
msgpack::object obj = msg.get();
obj.convert(&outputString);

std::cout << outputString << std::endl;

pac.next(&msg);
obj = msg.get();
obj.convert(&outputVector);

std::cout << outputVector[0] << ", " << outputVector[1] << std::endl;

pac.next(&msg);
obj = msg.get();
obj.convert(&outputMap);

std::cout << outputMap["hello"] << std::endl;

{% endcodeblock %}

Below we create a type "Root" that can hold different types of data, and which data is going to be serialised is defined by the MSGPACK\_DEFINE macro. This could easily have been a struct with public variables instead.

{% codeblock custom msgpack class lang:cpp %}

// define custom msgpack serialisation for class
class Root
{
private:
    std::vector<int> ints_;
    std::string string_;
public:
    std::vector<int> getInts()
    {
        return ints_;
    };
    std::string getString()
    {
        return string_;
    };
    Root(){};
    Root(const std::vector<int> &ints, const std::string &string): ints_(ints), string_(string){};

    // msgpack macro to define what to serialize in custom classes
    MSGPACK_DEFINE(ints_, string_);
};

{% endcodeblock %}

The code to use the custom serialisation is very similar to the simple example above:

{% codeblock custom msgpack lang:cpp %}

// ---- create the structure that will be serialised ----
std::vector<int> ints;
ints.push_back(2);
ints.push_back(5);

Root fromScratch(ints, "hello");

// ---- serialize it into buffer ----
msgpack::sbuffer sbuf;
msgpack::pack(sbuf, fromScratch);

// ---- deserialize it ----
msgpack::unpacked msg;
msgpack::unpack(&msg, sbuf.data(), sbuf.size());

msgpack::object obj = msg.get();

// ---- convert msgpack objects into statically typed object ----
Root world;
obj.convert(&world);

std::cout << world.getInts()[0] << ", " << world.getInts()[1] << std::endl;
std::cout << world.getString() << std::endl;

{% endcodeblock %}

Conclusion
----------

In the world of dynamic languages msgpack seems like a very good choice for use with messaging. I haven't gone through it here, but it can easily convert the dynamic languages standard containers (maps and arrays) into a msgpack binary message without too much problem, and in a really effecient way. The problem with the use of this library with static languages is that it requires hacks and quite a lot of boilerplate code to get around the static limitation that appear when message structure becomes more complicated.

I think c++ is where protobuf really shines. It provides a very simple api to a complicated problem as well as a list of interesting features such as default values, extensions and message validation and the only limitation is the fact that protobuf schemas (and the respective generated code) have to stay consistent wherever the messaging is used.

Saying that, if the message structure that you wish to send from the c++ program is a simple std::map or std::vector using PODs then I see no reason why to choose protobuf over msgpack.
