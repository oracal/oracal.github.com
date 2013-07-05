---
layout: post
title: "Xml C++ Examples"
slug: "xml-c++-examples"
date: 2013-06-28 06:55
comments: true
categories:
- c++
- xml
tags:
- c++
- xml
---

In each of the examples below I have used the address example from the previous article as an example xml file to be read in and parsed (as is common with xml, especially configuration files), create an xml message from scratch, query the xml message and also output as xml to a stream or file. I have separated out the querying into an outputAddress function that will be used throughout the example.

<!-- more -->

### Xsd

- uses a data binding technique with generated code from an xsd schema file
- a very small amount of code is required to effectively use the library
- requires an xsd
- uses generated code, which will need to be regenerated every time there is a schema change
- simple to use
- restrictive license

{% codeblock xsd.cpp lang:cpp %}

// ---- parse from file ----

boost::shared_ptr<Address> parsedFromFile;
try
{
    // read directly from file or uri
    parsedFromFile = Address_("address.xml");

    outputAddress(*parsedFromFile);
}
// all exceptions inherit from xml_schema::exception, can catch more specific if required
catch (const xml_schema::exception& e)
{
    std::cout << e.what() << std::endl;
}

// ---- create from scratch ----

// the constructor takes in the amount of required fields in the xsd as arguments
Address fromScratch("Mr. Malcolm Reynolds",
        "3",
        "Serenity",
        "Space",
        "DE18 5HI");

// set values
fromScratch.County("Solar System");
fromScratch.Country("UK");

std::stringstream ss;

// ---- parse from stream ----

xml_schema::namespace_infomap map;
map[""].schema = "address.xsd";

Address_(ss, fromScratch, map);

boost::shared_ptr<Address> parsedFromStream(Address_(ss));

outputAddress(*parsedFromStream);

// ---- output to file ----

std::ofstream ofs ("test.xml");
Address_(ofs, *parsedFromStream, map);

{% endcodeblock %}

{% codeblock xsd.cpp lang:cpp %}

void outputAddress(const Address& address)
{
    // output values using methods matching members in the xsd
    std::cout << address.Recipient() << std::endl;
    std::cout << address.House() << " " << address.Street() << std::endl;
    std::cout << address.Street() << std::endl;
    std::cout << address.Country() << std::endl;
    std::cout << address.PostCode() << std::endl;
}

{% endcodeblock %}

### Rapidxml

- very fast
- header only
- verbose api, but usable

{% codeblock rapidjson.cpp lang:cpp %}

// ---- parse from file and stream ----

// need to convert file and stream to cstring before parsing as rapidxml needs a null terminated cstring for parsing

// file to string
std::ifstream fin("address.xml");
std::stringstream ss;
ss << fin.rdbuf();
std::string xml = ss.str();

// string to dynamic cstring
std::vector<char> stringCopy(xml.length(), '\0');
std::copy(xml.begin(), xml.end(), stringCopy.begin());
char *cstr = &stringCopy[0];

// create xml document object and parse cstring
// character type defaults to char
rapidxml::xml_document<> parsedFromFile;
// 0 means default parse flags
try
{
    parsedFromFile.parse<0>(cstr);

    rapidxml::xml_node<> *addressNode = parsedFromFile.first_node("Address");
    outputAddress(*addressNode);

    // Print to stream using operator <<
    std::cout << parsedFromFile;

    // Print to stream using print function, specifying printing flags
    // 0 means default printing flags
    rapidxml::print(std::cout, parsedFromFile, 0);
}
catch(const rapidxml::parse_error &e)
{
    std::cout << "Parse error due to " << e.what() << std::endl;
}

// ---- create from scratch ----

rapidxml::xml_document<> fromScratch;
rapidxml::xml_node<> *addressNode = fromScratch.allocate_node(rapidxml::node_element, "Address");
rapidxml::xml_node<> *recipientNode = fromScratch.allocate_node(rapidxml::node_element, "Recipient", "Mr. Malcolm Reynolds");
rapidxml::xml_node<> *houseNode = fromScratch.allocate_node(rapidxml::node_element, "House", "3");
rapidxml::xml_node<> *streetNode = fromScratch.allocate_node(rapidxml::node_element, "Street", "Serenity");
rapidxml::xml_node<> *townNode = fromScratch.allocate_node(rapidxml::node_element, "Town", "Space");
rapidxml::xml_node<> *postCodeNode = fromScratch.allocate_node(rapidxml::node_element, "PostCode", "DE18 5HI");

addressNode->append_node(recipientNode);
addressNode->append_node(houseNode);
addressNode->append_node(streetNode);
addressNode->append_node(townNode);
addressNode->append_node(postCodeNode);

fromScratch.append_node(addressNode);

outputAddress(*addressNode);

// ---- output to file ----

std::ofstream fout("test.xml");
fout << fromScratch;

{% endcodeblock %}

{% codeblock rapidjson.cpp lang:cpp %}

void outputAddress(const rapidxml::xml_node<> &addressNode)
{
    std::cout << addressNode.first_node("Recipient")->value() << std::endl;
    std::cout << addressNode.first_node("House")->value() << " " << addressNode.first_node("Street")->value() << std::endl;
    std::cout << addressNode.first_node("Town")->value() << std::endl;
    std::cout << addressNode.first_node("PostCode")->value() << std::endl;
    std::cout << addressNode.first_node("Country")->value() << std::endl;

}

{% endcodeblock %}

### Tinyxml

- a lot of tiny memory allocations, so can be rather slow
- simple api for simple tasks, otherwise seems quite complicated

{% codeblock tinyxml.cpp lang:cpp %}

// ---- parse from file ----

TiXmlDocument parsedFromFile( "address.xml" );
parsedFromFile.LoadFile();

outputAddress(parsedFromFile.FirstChildElement("Address")->ToElement());

// ---- create from scratch ----

TiXmlDocument * fromScratch = new TiXmlDocument();

TiXmlElement * address = new TiXmlElement( "Address" );
TiXmlElement * recipient = new TiXmlElement( "Recipient" );

// LlinkEndChild causes memory managed object to be owned by the parent object
// and destroyed by the parent when destroyed
recipient->LinkEndChild(new TiXmlText("Mr. Malcolm Reynolds"));
address->LinkEndChild(recipient);

TiXmlElement * house = new TiXmlElement( "House" );
house->LinkEndChild(new TiXmlText("3"));
address->LinkEndChild(house);
TiXmlElement * street = new TiXmlElement( "Street" );
street->LinkEndChild(new TiXmlText("Serenity"));
address->LinkEndChild(street);
TiXmlElement * town = new TiXmlElement( "Town" );
town->LinkEndChild(new TiXmlText("Space"));
address->LinkEndChild(town);
TiXmlElement * postCode = new TiXmlElement( "PostCode" );
postCode->LinkEndChild(new TiXmlText("DE18 5HI"));
address->LinkEndChild(postCode);
TiXmlElement * country = new TiXmlElement( "Country" );
country->LinkEndChild(new TiXmlText("UK"));
address->LinkEndChild(country);

fromScratch->LinkEndChild( address );

outputAddress(fromScratch->FirstChildElement("Address")->ToElement());

// ---- parse from string ----

std::stringstream ss;
ss << *fromScratch;

TiXmlDocument parsedFromString;

parsedFromString.Parse(ss.str().c_str());

outputAddress(parsedFromString.FirstChildElement("Address")->ToElement());

// ---- output to file ----

parsedFromString.SaveFile("test.xml");

// destroying everything above that has been dynaimcally allocated by
// destorying the root node
delete fromScratch;

{% endcodeblock %}

{% codeblock tinyxml.cpp lang:cpp %}

void outputAddress(const TiXmlElement * address)
{
    std::cout << address->FirstChildElement("Recipient")->FirstChild()->Value() << std::endl;
    std::cout << address->FirstChildElement("House")->FirstChild()->Value() << " " << address->FirstChildElement("Street")->FirstChild()->Value() << std::endl;
    std::cout << address->FirstChildElement("Town")->FirstChild()->Value() << std::endl;
    std::cout << address->FirstChildElement("PostCode")->FirstChild()->Value() << std::endl;
    std::cout << address->FirstChildElement("Country")->FirstChild()->Value() << std::endl;
}

{% endcodeblock %}

### Pugixml

- very simple api
- reasonably fast

{% codeblock pugixml.cpp lang:cpp %}

// ---- parse from file ----

pugi::xml_document parsedFromFile;

pugi::xml_parse_result result = parsedFromFile.load_file("address.xml");

if(result)
{
    outputAddress(parsedFromFile.child("Address"));
}

// ---- create from scratch ----

pugi::xml_document fromScratch;
pugi::xml_node address = fromScratch.append_child("Address");

address.append_child("Recipient").append_child(pugi::node_pcdata).set_value("Mr. Malcolm Reynolds");
address.append_child("House").append_child(pugi::node_pcdata).set_value("3");
address.append_child("Street").append_child(pugi::node_pcdata).set_value("Serenity");
address.append_child("Town").append_child(pugi::node_pcdata).set_value("Space");
address.append_child("PostCode").append_child(pugi::node_pcdata).set_value("DE18 5HI");
address.append_child("County").append_child(pugi::node_pcdata).set_value("Solar System");
address.append_child("Country").append_child(pugi::node_pcdata).set_value("UK");

outputAddress(address);

// ---- parse from stream ----

// output document as xml to stream
std::stringstream ss;
fromScratch.save(ss);

pugi::xml_document parsedFromStream;

pugi::xml_parse_result streamResult = parsedFromStream.load(ss);

if(streamResult)
{
    outputAddress(parsedFromStream.child("Address"));

    // ---- output to file ----

    std::ofstream ofs ("test.xml");
    parsedFromStream.save(ofs);
}

{% endcodeblock %}

{% codeblock pugixml.cpp lang:cpp %}

void outputAddress(const pugi::xml_node& address)
{
    std::cout << address.child_value("Recipient") << std::endl;
    std::cout << address.child_value("House") << " " << address.child_value("Street") << std::endl;
    std::cout << address.child_value("Town") << std::endl;
    std::cout << address.child_value("PostCode") << std::endl;
    std::cout << address.child_value("Country") << std::endl;
}

{% endcodeblock %}

### Boost Property Tree

- header only
- comes packaged with boost
- uses rapidxml to parse xml
- simple to use api

{% codeblock boost.cpp lang:cpp %}

// ---- parse from file ----

// every elelment is a ptree
boost::property_tree::ptree parsedFromFile;

try
{
    // use trim whitespace to remove any whitespace due to pretty formatting of xml structure
    read_xml("address.xml", parsedFromFile, boost::property_tree::xml_parser::trim_whitespace);

    outputAddress(parsedFromFile);

}
catch(const boost::property_tree::xml_parser::xml_parser_error &e)
{
    std::cout << "Could not parse file due to " << e.what() << std::endl;
}

// ---- create from scratch ----

boost::property_tree::ptree fromScratch;
boost::property_tree::ptree address;

// replace put with add if element key is not guaranteed to be unique
address.put("Recipient", "Mr. Malcolm Reynolds");
address.put("House", "3");
address.put("Street", "Serenity");
address.put("Town", "Space");
address.put("PostCode", "DE18 5HI");
address.put("County", "Solar System");
address.put("Country", "UK");
fromScratch.put_child("Address", address);

outputAddress(fromScratch);

// ---- parse from stream ----

// write and read methods can take paths to filenames and streams as parameters
std::stringstream ss;
write_xml(ss, fromScratch);

boost::property_tree::ptree parsedFromStream;

read_xml(ss, parsedFromStream);

outputAddress(parsedFromStream);

// ---- output to file ----

std::ofstream fout("test.xml");
write_xml(fout, parsedFromStream);

{% endcodeblock %}

{% codeblock boost.cpp lang:cpp %}

void outputAddress(const boost::property_tree::ptree &pt)
{
    // can use dotted notation to specify element path
    std::stringstream ss;
    try
    {
        ss << pt.get<std::string>("Address.Recipient") << std::endl;
        ss << pt.get<std::string>("Address.House") << " " << pt.get<std::string>("Address.Street") << std::endl;
        ss << pt.get<std::string>("Address.Town") << std::endl;
        ss << pt.get<std::string>("Address.PostCode") << std::endl;
        ss << pt.get<std::string>("Address.Country") << std::endl;
        std::cout << ss.str();
    }
    catch(const boost::property_tree::ptree_bad_path &e)
    {
        std::cout << e.what() << std::endl;
    }
}

{% endcodeblock %}

Conclusion
----------

I think the choice is very much dependent on what your use case is. If you have an xsd and it is small enough to allow free use of the xsd library (under the terms of their license) then I think that it is a very attractive option to consider.

Alternatively pugixml has a very simple api and is relatively fast, and so is an obvious choice, especially if there is a package available on your distribution that you can link against.

If you are looking for super fast encoding/decoding of xml then your two options are rapidxml and boost property tree. The boost library uses rapidxml to parse xml and therefore should be very comparable in speed. The good thing about boost is the fact that it allows a convenient package management solution for a header only library and boost is a very common requirement for a lot of projects, so in most cases you will already have access to the xml library, as well as slightly improving the rapidxml api.

