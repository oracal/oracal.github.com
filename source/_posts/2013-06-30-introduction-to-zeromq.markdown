---
layout: post
title: "Introduction to Zeromq"
slug: "introduction-to-zeromq"
date: 2013-12-20 11:43
comments: true
categories:
- c++
- networking
---

In the modern world there are rarely any stand alone applications anymore. Everything is connected. Whether it be an android application calling a restful api or a webpage connecting to thousands of internet users. When working with large systems there is always a need for communication between connected components. Web services are often a great way to communicate with each other and allow a single restful api to control all componenets. However, sometimes we need faster communication between components. This is needed so often that it seems weird that everytime it is needed people seem to roll their own, ussualy some some awkward messaging format using a custom wrapper around the socket api (more often than not an OS dependent one).

<!-- more -->

Over the Wire
-------------

The socket api is fundamental to network communication, it is the standard that has existed for many years, it provides a file like api to network commnications which abstracts the complications of network transport. It is very useful and powerful, but it is too low-lovel. There are too many edge cases in which you must protect yourself against. Many people create custom wrappers around the socket api to deal with these edge cases. In c++ there is the boost asio library which abstracts the socket api, but not by much (in fact it still uses sockets, it just makes them platform independent). These custom wrappers never catch everything and often take quite a bit of time to create.

Why not use a library created by some of the experts in the field and which deals with a lot of these edge cases for you?

Enter zeromq. Zeromq provides a very powerful abstraction on communication, which allows networked applications to talk to each other very easily and resiliently. The blurb from the developers themselves provides a very good description of what zeromq is and what it does:

>ØMQ (also seen as ZeroMQ, 0MQ, zmq) looks like an embeddable networking library but acts like a concurrency framework. It gives you sockets that carry atomic messages across various transports like in-process, inter-process, TCP, and multicast. You can connect sockets N-to-N with patterns like fanout, pub-sub, task distribution, and request-reply. It's fast enough to be the fabric for clustered products. Its asynchronous I/O model gives you scalable multicore applications, built as asynchronous message-processing tasks. It has a score of language APIs and runs on most operating systems. ØMQ is from iMatix and is LGPLv3 open source.

Now that I have given you a brief introduction lets see what this library can do.

Implementation
--------------

The first example I will show you is the most simplest possible. In fact this could easily be done using sockets. It is a simple request and response between a client and server. Specifically in this example I pass a msgpack binary message over the zeromq connection.

A couple of c++ implementation details to note is that all data that zeromq passes around inside messages uses void pointers, and these need to be staticaly cast into the type of data that you actually want (msgpack uses char \*, but a lot of the time these will be converted into std::strings, I'll do this plenty of times in my json examples). Also since everything that is sent over zeromq is formed of messages we must populate zmq::message\_t objects and send them using the zeromq socket api. One final thing to note down is that zeromq needs to work with multiple languages including ones that treat strings differently, namely null terminating string compared to most other types of strings. To keep things consistent zeromq has decided to not send null terminating strings so this is a very important factor to keep in mind when dealing with languages that use null terminating strings such as c++.

Request Reply
-------------

The client below starts by generating some msgpack data, then it creates a zeromq context, this is required for an application to use zeromq and starts off all the asyncrounous fun that zeromq uses behind the scenes. The number argument that you pass to this context is the amount of threads that zeromq will use in the background and should obviously be optimised for your application/hardware. Next we create a zmq socket that has a very similar api to a normal socket, however we pass to it an ENUM which describes the scenario that we would like zeromq to use, in this case the request part of a request-reply scenario. There are quite a few scenarios and I will go over a few of them in this post.

Once the socket has been created it needs to be connected to an end point. In this case the server application that will be shown below. It is important to note that the server doesn't neccesarily need to be started first for the client to connect to it. Due to the way way zeromq buffers messages either one can be started first.

For each piece of data that we send to the socket we are required to create a zmq::message\_t object which we can populate with any data cast to a void \* pointer. When we create a zmq message we must pass into it the amount of bytes that message will be in the constructor. We can then memcpy the data over to it.

Then it's just a matter of sending over the message and receiving a reply message.

{% codeblock lang:cpp %}

// the vector that is going to be sent
std::vector<std::string> vec;
vec.push_back("Hello");
vec.push_back("MessagePack");

// serialize it into simple buffer.
msgpack::sbuffer sbuf;
msgpack::pack(sbuf, vec);

//  Prepare our context and socket
zmq::context_t context(1);
zmq::socket_t socket(context, ZMQ_REQ);

socket.connect("tcp://localhost:5555");

//  Do 10 requests, waiting each time for a response
for (int request_nbr = 0; request_nbr != 10; request_nbr++)
{
    // create request object, fill it up and send it
    zmq::message_t request(sbuf.size());
    memcpy(request.data(), sbuf.data(), sbuf.size());
    socket.send(request);

    //  Get the reply.
    zmq::message_t reply;
    socket.recv(&reply);
}

{% endcodeblock %}

Now for the server, it sets things up in a very similar way to the client, except that this time we use the reply ENUM ZMQ\_REP, and we bind the server to a port rather than to an ip address.

This time we create a zmq::message\_t object to receive the incoming message. We do not pass a number to the constructor of the message object this time since obviously we don't know yet what that is and also since when we pass it into the socket.recv() method the size will be populated with the incoming message size. In the example we then decode the msgpack message and turn it into a static type object and then send a reply back, all very easy stuff.

{% codeblock lang:cpp %}

//  Prepare our context and socket
zmq::context_t context(1);
zmq::socket_t socket(context, ZMQ_REP);
socket.bind("tcp://*:5555");

while(true)
{
    zmq::message_t request;

    //  Wait for next request from client
    socket.recv(&request);
    msgpack::sbuffer sbuf;
    sbuf.write(static_cast<const char *>(request.data()), request.size());

    // deserialize it.
    msgpack::unpacked msg;
    msgpack::unpack(&msg, sbuf.data(), sbuf.size());

    // print the deserialized object.
    msgpack::object obj = msg.get();
    std::cout << obj << std::endl;

    // convert it into statically typed object.
    std::vector<std::string> rvec;
    obj.convert(&rvec);

    // Do some 'work'
    sleep (1);

    //  Send reply back to client
    zmq::message_t reply(11);
    memcpy(reply.data(), "Hello World", 11);
    socket.send(reply);
}

{% endcodeblock %}

Publish and Subscribe
---------------------

The publish and subscribe pattern is quite a common network structure and involves a single server constantly outputting data and clients subscribing to the server so that they too receive the same information from the server. If just using sockets for this problem alot of extra code would have to be written to accomodate this new architecture, having to deal with the subscriptions of all clients and making sure that edge cases of this system would not cause a problem.

Below is an example of a weather server sending out important weather information. It is very similar to the example given in the Zeromq documentation, however I have replaced the data being sent to a json representation. You will notice that underlying zeromq code for this is very similar to the response and reply examples above, with the only real difference being the second argument to the socket constructor. The enum ZMQ\_PUB defines the publisher socket type and means that all the extra complexities of using a publisher subscriber pattern will now be dealt with under the zeromq socket api. And similarly with the client example below the ZMQ\_SUB enum is passed in allowing the specific socket code for subscriptions to be applied.

One extra detail in this example is that there is an envelope message which is supposed to define the type of message being sent, this uses two separate zeromq messages but uses the ZMQ\_SNDMORE argument to the send method (part of the zeromq api). This means that the message will only be sent the next time a send without ZMQ\_SNDMORE argument is called. This allows you to combine separate zeromq messages into single "real messages". In this example it is used to filter out certain messages from the publisher.

{% codeblock lang:cpp %}

#define within(num) (int) ((float) num * random () / (RAND_MAX + 1.0))

//  Prepare our context and publisher
zmq::context_t context(1);
zmq::socket_t publisher(context, ZMQ_PUB);
publisher.bind("tcp://*:5556");

Json::Value root;

//  Initialize random number generator
srandom((unsigned) time(NULL));

while(1)
{
    int zipcode, temperature, relhumidity;

    //  Get values that will fool the boss
    zipcode     = within(100000);
    temperature = within(215) - 80;
    relhumidity = within(50) + 10;
    root["zipcode"] = zipcode;
    root["temperature"] = temperature;
    root["relhumidity"] = relhumidity;

    Json::StyledWriter styledWriter;
    std::cout << styledWriter.write(root);

    Json::FastWriter fastWriter;
    std::string jsonMessage = fastWriter.write(root);

    //  Send message to all subscribers
    zmq::message_t message(jsonMessage.length());
    zmq::message_t filter(4);
    memcpy(filter.data(), "test", 4);

    memcpy(message.data(), jsonMessage.c_str(), jsonMessage.length());
    // send a message that will wait for other messages to send using flag
    publisher.send(filter, ZMQ_SNDMORE);

    publisher.send(message);
    sleep(1);
}

{% endcodeblock %}

The client example below is very similar to the client in the request reply example above.

{% codeblock lang:cpp %}

// set up jsoncpp objects
Json::Value v;
Json::Reader reader;
Json::StyledWriter styledWriter;

// set up zeromq context
zmq::context_t context(1);

//  Socket to talk to server
std::cout << "Collecting updates from server..." << std::endl;
zmq::socket_t subscriber(context, ZMQ_SUB);
subscriber.connect("tcp://localhost:5556");

// subscribe to filter
std::string filter = "test";
subscriber.setsockopt(ZMQ_SUBSCRIBE, filter.c_str(), filter.length());

//  Process 100 updates
int update_nbr;
for(update_nbr = 0; update_nbr < 100; update_nbr++)
{
    zmq::message_t envelope;
    zmq::message_t update;
    // receive the envelope message which is used to filter out subscribers
    subscriber.recv(&envelope);
    // receive the json message and actually do soemthing with it
    subscriber.recv(&update);
    // convert non null terminating cstring into a string
    std::string response = std::string(
            static_cast<const char*>(update.data()),
            update.size());

    // parse json data
    bool parsingSuccessful = reader.parse(response, v);
    if(parsingSuccessful)
    {
        std::cout << styledWriter.write(v);
    }
}

{% endcodeblock %}

Conclusion
----------

I have just given some basic examples of using zeromq and how easy it is to use. Zeromq has quite a few different network patterns at its disposal and many of them are much more complex, however, they all have a similar simple api to use (the hard part is understanding the network pattern itself). What I am mainly interested in is the use of zeromq in edge-cases, such as network connectivity issues; I believe it will allow me to write much more fault-tolerant software in the future. I look forward to using zeromq in real applications and learning about the other network patterns available.
