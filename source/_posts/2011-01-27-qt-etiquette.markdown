---
comments: true
date: 2011-01-27 20:55:41
layout: post
slug: qt-etiquette
title: Qt Etiquette
categories:
- c++
- qt
---

So I have been using Qt now for about six months and I have to say it is one of the best frameworks I have ever used, it seems very complete, the documentation for it is amazing and even the Qt IDE: Qt Creator is awesome. Some of the really useful things I find are the signal and slot mechanism, the widget methodology and the expanse of widgets available. So I feel pretty confident with Qt now and just though I'd share some information that I learned along the way that may or may not be included in the documentation, the sort of info only found by experience. Hopefully this helps a few people out, so here it goes.

<!-- more -->

One thing I found when using signals and slots was that I had trouble connecting signals and slots when they there was a widget inside a widget inside a widget, lets call them A, B and C respectively. Setting up a connection between A and C was impossible, what I had to do was create an intermediate slot inside B that emitted a signal from B, to keep the signal going. This was pretty annoying as I had to create the new slot function and there was a simple solution that I didn't get for a while. Connections can be made between a signal and a signal, so that when the first emitted so is the other signal, now this may not be a huge improvement in code, but it looks so much neater and when working on a larger project it is so much easier to follow. A simple example, not sure it's really needed, but anyway here it is:

{% codeblock lang:cpp %}
connect(myButton, SIGNAL(clicked()), this, SIGNAL(buttonClicked()));
{% endcodeblock %}

Another problem I had with signals and slots was that sometimes I'd like to quickly take control of my connection and stop it for a while and then reconnect it, and this was fine I could use the

{% codeblock lang:cpp %}QObject:disconnect{% endcodeblock %}

method to explicitly remove the connection and then reattach it in the normal way. This gets a bit complicated when you are dealing with an important QObject and there a lot of connections that need to be disconnected, half your code would be establishing and disconnecting connections. There's a handy method in the QObject toolbox that solves this problem called

{% codeblock lang:cpp %}QObject:blockSignals(){% endcodeblock %}

This function blocks all signals being emitted from an object, if set to true and you can easily remove this block by calling this function again with the false parameter.

These two fixes really helped me fall in love with the signal and slot mechanism behind pretty much any Qt program.

A lot of questions I see paraded around in regards to Qt is about which containers classes to use. Now I have pretty much been brought up on the STL, and know them pretty well but I am a big believer that when using a particular framework that you use the more integrated containers. Not only that but the documentation for the Qt containers insdie the Qt Creator IDE is impressive, press F1 and I have every single meber function and there are a lot of them. Not only that but reading some of the documentation surrounding Qt containers brought me to an underlying methodology called implicitly shared. Which pretty much means that within any Qt container class it will make your code more efficient by reducing the copying with that container, vist the [website](http://doc.qt.nokia.com/latest/implicit-sharing.html#implicitly-shared) for more details. Also lots of outputs of other functions within Qt generate the Qt generic classes so the amount of code needed to convert and reconvert would be a bit excessive.

Another thing I liked about Qt was how easy it was to access external applicationsand give input arguemnts and easily receive standard outputs (mainly command line). This has been really useful when making wrappers for some of my commonly used command line applications (a task I try to do whenever learning a new gui framework) this meant I didn't have to worry too much about looking through a lot of code and the only way to proceed when using a closed source application. A simple example:

{% codeblock lang:cpp %}
// declare a new QProcess
QProcess *extraction = new QProcess;

// define command line arguments
QStringList args;
args << "e" << file << "/ad" << "/y";

// run external application
extraction->execute("C:/Program Files/WinRAR/unRAR", args);

// get the output from the application
QByteArray output = extraction->readAllStandardOutput();
{% endcodeblock %}

I hope this has been at least slightly helpful to anyone who has read it, and for anyone who has yet to make the leap into the Qt framework I say go for it, you won't regret it.
