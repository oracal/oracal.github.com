---
layout: post
title: "Json C++ Examples"
slug: "json-c++-examples"
date: 2013-06-27 06:54
comments: true
categories:
- c++
- json
tags:
- c++
- json
---

In each of the examples below I have tried to show you most of the different aspects of using json, i.e. creating json messages from scratch, outputting json, parsing json and querying json objects. The example json message used contains most of the features that a json message could contain. Querying the json object has been extracted into an output function which is used multiple times in each example to show that everything is working correctly.

<!-- more -->

### jsoncpp

- mature
- feature complete
- c++ interface
- very simple api

{% codeblock jsoncpp.cpp lang:cpp %}

// ---- create from scratch ----

Json::Value fromScratch;
Json::Value array;
array.append("hello");
array.append("world");
fromScratch["hello"] = "world";
fromScratch["number"] = 2;
fromScratch["array"] = array;
fromScratch["object"]["hello"] = "world";

output(fromScratch);

// write in a nice readible way
Json::StyledWriter styledWriter;
std::cout << styledWriter.write(fromScratch);

// ---- parse from string ----

// write in a compact way
Json::FastWriter fastWriter;
std::string jsonMessage = fastWriter.write(fromScratch);

Json::Value parsedFromString;
Json::Reader reader;
bool parsingSuccessful = reader.parse(jsonMessage, parsedFromString);
if (parsingSuccessful)
{
    std::cout << styledWriter.write(parsedFromString) << std::endl;
}

{% endcodeblock %}

{% codeblock jsoncpp.cpp lang:cpp %}

void output(const Json::Value & value)
{
    // querying the json object is very simple
    std::cout << value["hello"];
    std::cout << value["number"];
    std::cout << value["array"][0] << value["array"][1];
    std::cout << value["object"]["hello"];
}

{% endcodeblock %}

### rapidjson

- header only
- very fast
- c++ interface
- verbose api

{% codeblock rapidjson.cpp lang:cpp %}

// ---- create from scratch ----

// document is the root of a json message
rapidjson::Document fromScratch;

// define the document as an object rather than an array
fromScratch.SetObject();

// create a rapidjson array type with similar syntax to std::vector
rapidjson::Value array(rapidjson::kArrayType);

// must pass an allocator when the object may need to allocate memory
rapidjson::Document::AllocatorType& allocator = fromScratch.GetAllocator();

// chain methods as rapidjson provides a fluent interface when modifying its objects
array.PushBack("hello", allocator).PushBack("world", allocator);

fromScratch.AddMember("hello", "world", allocator);
fromScratch.AddMember("number", 2, allocator);
fromScratch.AddMember("array", array, allocator);

// create a rapidjson object type
rapidjson::Value object(rapidjson::kObjectType);
object.AddMember("hello", "world", allocator);
fromScratch.AddMember("object", object, allocator);
fromScratch["object"]["hello"] = "world";

output(fromScratch);

// ---- parse from string ----

// Convert JSON document to string
rapidjson::StringBuffer strbuf;
rapidjson::Writer<rapidjson::StringBuffer> writer(strbuf);
fromScratch.Accept(writer);

// parse json string
rapidjson::Document parsedFromString;
parsedFromString.Parse<0>(strbuf.GetString());

output(parsedFromString);

{% endcodeblock %}

{% codeblock jansson.cpp lang:cpp %}

void output(const rapidjson::Document & document)
{
    // treat object types similar to std::map when querying
    std::cout << document["hello"].GetString() << std::endl;
    std::cout << document["number"].GetInt() << std::endl;

    // requires SizeType since the literal zero in c++ can mean a
    // numeric type (int, unsigned, etc.) or a null pointer of any type
    std::cout << document["array"][rapidjson::SizeType(0)].GetString() << std::endl;

    std::cout << document["array"][1].GetString() << std::endl;

    std::cout << document["object"]["hello"].GetString() << std::endl;
}

{% endcodeblock %}

### jansson

- mature
- simple c interface
- nasty memory allocation mainly due to it being a c library
- every json item is a json\_t pointer, which must be checked to see what type it is before being used

{% codeblock jansson.cpp lang:cpp %}

// ---- create from scratch ----

// create json "objects" and use the json_t pointer to point to them
json_t *fromScratch = json_object();
json_t *array = json_array();

// use the new version of the method to aboud have to call json_decref on each json object
// must encompass values in their respective methods
json_array_append_new(array, json_string("hello"));
json_array_append_new(array, json_string("world"));

json_object_set_new(fromScratch, "hello", json_string("world"));
json_object_set_new(fromScratch, "number", json_integer(2));
json_object_set_new(fromScratch, "array", array);

// json pack provies an easy to use api to create json structures using a
// special string syntax as the first argument
json_object_set_new(fromScratch, "object", json_pack("{ss}", "hello", "world"));

output(fromScratch);

// ---- parse from string ----

// use json_dumps to output raw json from the json "objects"
char * jsonOutput = json_dumps(fromScratch, 0);

// can use special flags to make the json more human readible
std::cout << json_dumps(fromScratch, JSON_INDENT(2)) << std::endl;

json_error_t error;

// parse the json and
json_t *parsedFromString = json_loads(jsonOutput, 0, &error);

// memory management for the json "objects"
free(jsonOutput);
json_decref(fromScratch);

if(parsedFromString)
{
    output(parsedFromString);
}
else
{
    std::cout << error.text << std::endl;
}

json_decref(parsedFromString);

{% endcodeblock %}

{% codeblock jansson.cpp lang:cpp %}

void output(const json_t* document)
{
    // lots of checking to see if json_t is of the right type
    // since it is just a json_t pointer
    if(json_is_object(document))
    {
        json_t* string = json_object_get(document, "hello");
        if(json_is_string(string))
        {
            std::cout << json_string_value(string) << std::endl;
        }
        json_t* number = json_object_get(document, "number");
        if(json_is_integer(number))
        {
            std::cout << json_integer_value(number) << std::endl;
        }
        json_t* array = json_object_get(document, "array");

        if(json_is_array(array))
        {
            json_t* index;
            index = json_array_get(array, 0);
            if(json_is_string(index))
            {
                std::cout << json_string_value(index) << std::endl;
            }
            index = json_array_get(array, 1);
            if(json_is_string(index))
            {
                std::cout << json_string_value(index) << std::endl;
            }
        }

        json_t* object = json_object_get(document, "object");
        if(json_is_object(object))
        {
            json_t* objectString = json_object_get(object, "hello");
            if(json_is_string(objectString))
            {
                std::cout << json_string_value(objectString) << std::endl;
            }
        }
    }
}

{% endcodeblock %}

Conslusion
----------

I think the choice is very much dependent on what your use case is. If you would like a very simple api and do not rely too heavily on being super fast then the obvious choice is jsoncpp, especially if there is a package available on your distribution that you can link against. If you require super fast json encoding and decoding or are working on a system where a header-only library is an attractive option then choose rapidjson. Since we're talking about c++ here, I don't think there should be a situation in which you should choose jansson over the others.
