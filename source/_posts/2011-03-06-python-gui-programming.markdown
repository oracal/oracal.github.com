---
comments: true
date: 2011-03-06 21:28:00
layout: post
slug: python-gui-programming
title: Python GUI Programming
categories:
- python
- gui
- pygtk
- pyqt
- pyside
- wxpython
---

There are many options for GUI programming with Python. I'll go over my favorite, and generally most well known frameworks and show you some simple programs with each one. In fact these aren't really Python frameworks but actually Python bindings for already established c/c++ libraries. So the three options are PyQt/PySide, wxPython  and PyGTK with their respective c++ frameworks Qt, wxWidgets and GTK+. I'll go over the differences of PyQt and PySide, I mention both since PyQt is more mature than PySide but has a more restrictive license, also PySide has just moved out of beta with its version 1 release, and according to its creators, the company behind the original Qt, it is ready for production level code.

<!-- more -->
The GUI program we'll make is a simple single button example that changes the color of a square rectangle. In each of the frameworks this will show its basic syntax, its widget creation and show its widget interconnectivity. Three of the most important aspects in using a GUI framework, especially widget connectivity.


## PyQt


When I first started looking at Python GUI frameworks I of course had already heard about PyQt from my extensive work with Qt using c++. If I ever found a solution online for a Qt problem using PyQt it was easily transferable to the c++ world. Now PyQt is the more mature set of bindings for Qt and therefore is generally a good choice if one already knows Qt, however it only has a GPL license which means you need to purchase a license to sell on the program when using PyQt. In contrast to PySide (which we shall discuss later) which has a LGPL license when can be used in a commercial application without license.

Now getting past the licensing issues, PyQt is extremely easy to work with and contains most of the functionality of the original Qt. I'm just going to show and explain an example bit of code that will get you used to the PyQt syntax and show some widget connectivity, which you should be able to adapt to any already implemented Qt widgets.

{% codeblock lang:python %}

import sys
from PyQt4.QtCore import *
from PyQt4.QtGui import *

class Form(QDialog):
	def __init__(self, parent=None):
		super(Form, self).__init__(parent)
		button=QPushButton("Change Color")
		self.label=QLabel(self)
		fill = QPixmap(20,20)
	 	fill.fill(Qt.red)
		self.label.setPixmap(fill)
		layout = QHBoxLayout()
		layout.addWidget(button)


		layout.addWidget(self.label)
		self.setLayout(layout)
		self.setWindowTitle("Color Changer")

		self.connect(button, SIGNAL("clicked()"), self.change)

	def change(self):
		fill = QPixmap(20,20)
		fill.fill(Qt.blue)
		self.label.setPixmap(fill)

app = QApplication(sys.argv)
form = Form()
form.show()
app.exec_()
{% endcodeblock %}



I'll go over each section of code to explain it



{% codeblock lang:python %}
from PyQt4.QtCore import *
from PyQt4.QtGui import *
{% endcodeblock %}



This bit of code simply imports the modules needed for the Qt framework.



{% codeblock lang:python %}
class Form(QDialog):
	def __init__(self, parent=None):
		super(Form, self).__init__(parent)
{% endcodeblock %}



This section of code creates our own widget that inherits the QDialog class and then defines the contructor which in turn calls the QDialog constructor.



{% codeblock lang:python %}
button=QPushButton("Change Color")
self.label=QLabel(self)
fill = QPixmap(20,20)
fill.fill(Qt.red)
self.label.setPixmap(fill)
self.setWindowTitle("Color Changer")
{% endcodeblock %}



Now still in the constructor we create a button and a label. We then fill the label with the color red. Fianlly we set the tite of the dialog.



{% codeblock lang:python %}
layout = QHBoxLayout()
layout.addWidget(button)
layout.addWidget(self.label)
self.setLayout(layout)
{% endcodeblock %}



This part of the code creates the layout of the dialog, pretty simple if you're coming from a Qt background.



{% codeblock lang:python %}
self.connect(button, SIGNAL("clicked()"), self.change)
{% endcodeblock %}



This is a really important bit here, this shows PyQt's implementation of the signals and slots mechanism. Here we connect the buttons clicked signal to the change function, which simply changes the color of the label.



{% codeblock lang:python %}
app = QApplication(sys.argv)
form = Form()
form.show()
app.exec_()
{% endcodeblock %}



Now bringing everything together, we create an QApplication oject which every Qt program needs. And then create an instance of our Form dialog and then call the function to show the dialog. We then enter the event loop of the application using {% codeblock lang:python %}app.exec_(){% endcodeblock %} (notice the slight difference with the original exec function in Qt to account for Python's own exec function), and thats it.





## PySide



Now this code is very similar to the PyQt code, as obviously it uses pretty much the same framework, so I'm only going to go over the changes in the code.



{% codeblock lang:python %}
import sys
from PySide.QtCore import *
from PySide.QtGui import *

class Form(QDialog):
		def __init__(self, parent=None):
		super(Form, self).__init__(parent)
		button=QPushButton("Change Color")
		self.label=QLabel(self)
		fill = QPixmap(20,20)
		fill.fill(Qt.red)
		self.label.setPixmap(fill)
		layout = QHBoxLayout()
		layout.addWidget(button)

		layout.addWidget(self.label)
		self.setLayout(layout)
		self.setWindowTitle("Color Changer")

		button.clicked.connect(self.change)


	def change(self):
		fill = QPixmap(20,20)
		fill.fill(Qt.blue)
		self.label.setPixmap(fill)

app = QApplication(sys.argv)
form = Form()
form.show()
app.exec_()
{% endcodeblock %}



Ok so for the differences:



{% codeblock lang:python %}
from PySide.QtCore import *
from PySide.QtGui import *
{% endcodeblock %}



We have a slightly different import section here for the new PySide bindings.



{% codeblock lang:python %}
button.clicked.connect(self.change)
{% endcodeblock %}



The signal and slots procedure is slightly different, here we just chain the object, the event and then the connect function with the function you want to be the slot inside the parenthesis.





So that's it, the only differences. In larger programs you would find a few more but for the most part they are identical.





## wxPython



This GUI framework is based on the c++ wxWidgets framework. It uses very similar code to the other examples above, with only slight differences in syntax, naming conventions and connecting widgets. We still create our own form that inherits from a dialog class, in this case wx.dialog. We then create the button, I'll go over the rest of the code after you have had a quick browse:
{% codeblock lang:python %}

import wx

class form(wx.Dialog):

	def __init__(self, parent, id, title):
		wx.Dialog.__init__(self, parent, id, title, size=(150, 60))
		self.button = wx.Button(self, -1, 'Change Color', (0, 0))

		self.panel  = wx.Panel(self, -1, (120, 2), (20, 20))
		self.panel.SetBackgroundColour('RED')

		self.Bind(wx.EVT_BUTTON, self.Change, self.button)

		self.Center()
		self.Show()

	def Change(self,event):
		self.panel.SetBackgroundColour('BLUE')
		self.panel.Refresh()

app = wx.App()
form(None, -1, 'Color Changer')
app.MainLoop()

{% endcodeblock %}



The code I want to draw your attention to is this bit here:



{% codeblock lang:python %}
self.Bind(wx.EVT_BUTTON, self.Change, self.button)
{% endcodeblock %}

This is the way wxPython connects its widgets together, using the Bind function and linking an event, in this case wx.EVT_BUTTON (which is just wxPython's button clicked event), of a certain button which calls a certain function self.change when activated.

{% codeblock lang:python %}
app = wx.App()
form(None, -1, 'Color Changer')
app.MainLoop()
{% endcodeblock %}



This code here creates an application object and then creates our own form class which inherits the wx.Dialog class and then calls app.MainLoop() to enter the event loop of the program




## PyGTK





Now I hate to say it but I'm really not a fan of PyGTK, but you should definitely know about it. Here's a bit of code that does the same as the other examples but I'm not going to explain it in detail because I'd suggest you look at either wxPython or one of the python Qt frameworks. You see the same general theme as the other frameworks, you import the specific module, and then create a window, add buttons using a layout object (in this case a HBox) and then connect widgets together (in this case the connect command chained onto the button object).

{% codeblock lang:python %}
import pygtk
import gtk

class form:

    def __init__(self):
        self.window = gtk.Window(gtk.WINDOW_TOPLEVEL)
        self.window.set_title("Color Changer")

        self.window.set_border_width(10)

        self.button = gtk.Button("Change Color")
        self.drawingarea = gtk.DrawingArea()
        self.drawingarea.set_size_request(30, 30)

        self.button.connect("clicked", self.show, None)

        self.hbox = gtk.HBox(False, 5)
        self.window.add(self.hbox)
        self.hbox.pack_start(self.button)
        self.hbox.pack_start(self.drawingarea)

        self.window.show_all()

        self.style = self.drawingarea.get_style()
        self.gc = self.style.fg_gc[gtk.STATE_NORMAL]
        self.gc.set_rgb_fg_color(gtk.gdk.Color(65535, 0, 0))
        self.drawingarea.connect("expose-event", self.area_expose_cb)

    def area_expose_cb(self, area, event):

        self.drawingarea.window.draw_rectangle(self.gc, True, 0, 0, 20, 20)

    def show(self, widget, data=None):
        self.gc.set_rgb_fg_color(gtk.gdk.Color(0, 0, 65535))
        self.drawingarea.window.draw_rectangle(self.gc, True, 0, 0, 20, 20)



hello = form()
gtk.main()
{% endcodeblock %}



As you might be able to tell I'm slightly biased towards Qt, but in this case I think I have some good reasons to be, not only by how simple the code is although that is a huge part of it, but also by the support from the developers and the community. For example the documentation on both the Qt frameworks is awesome and there are a lot of resources out there to learn from. Also they have adapted their Qt designer to work with the Python frameworks so one can design them physically and then use them in code. The other frameworks have this ability but it really is so much easier with Qt. Happy coding!
