---
title: JavaScript Scripting
section: Extend:Scripting:Languages
project: /software/imagej
---

{% include wikipedia title='JavaScript' text='JavaScript'%} is a high-level, dynamic, untyped programming language, supporting multiple paradigms including object-oriented, imperative and functional programming styles. Although there are similarities between JavaScript and [Java](/develop/plugins), including language name and syntax, the two are distinct languages and differ greatly in their design.

# A note about JavaScript engines

ImageJ supports JavaScript via Java's built-in {% include wikipedia title='Nashorn (JavaScript engine)' text='Nashorn JavaScript engine'%}. Versions of Java before Java 8 instead included Mozilla's {% include wikipedia title='Rhino (JavaScript engine)' text='Rhino JavaScript engine'%}. The two engines are largely, but not completely, compatible—meaning that some older scripts written for older versions of ImageJ (which used Rhino) may no longer function correctly when run with current ImageJ versions (which use Nashorn).

# JavaScript tutorial for ImageJ

## Language basics

### Importing classes

{% include warning/importing-classes lang='JavaScript' %} You can specify imports in JavaScript as follows:

```java
importClass(Packages.java.io.File)
```

Where `java.io.File` is the class to be imported.

For the most common stuff like e.g. *IJ*, the *RoiManager* or *GenericDialog* the following lines would be required:

```java
importClass(Packages.ij.IJ);
importClass(Packages.ij.plugin.frame.RoiManager);
importClass(Packages.ij.gui.GenericDialog);
```

#### Using the full name of a class

Sometimes it is easier to use the full name of a class than of using an import. E.g. you can use `Packages.ij.gui.GenericDialog` to access the class GenericDialog once.

For some packages there are build in shortcuts. You can use `java.io.File` instead of `Packages.java.io.File` as *java* is a shortcut for *Packages.java* (there are shortcuts for *Packages.java*, *Packages.javax*, *Packages.com*, *Packages.edu*, *Packages.javafx* and *Packages.org*).

### Variables

There are two ways:

-   Direct: the variable is globally visible (dangerous! Leads to nasty bugs.)
-   With a `var` declaration: the variable is local, visible only within the innermost code block.

**Local** variables are also **faster**, because they can be [efficiently optimized for access](http://www.mozilla.org/rhino/perf.html). Examples:

```java
importClass(Packages.ij.IJ);
// global variable 'imp'
imp = IJ.getImage();

// local variable 'i', visible within the loop only:
for (var i = 0; i < 10; i++ ) {
  IJ.log("i is " + i);
}
```

Mixing global and local variables in a sensible way:

```java
importClass(Packages.ij.IJ);
// global variables 'base_url' and 'names'
// and local variable 'imp', the latter visible within the loop only:

base_url = "https://imagej.nih.gov/ij/images/";

names = ["blobs.gif", "boats.gif", "bridge.gif"];

for (var i = 0; i< names.length; i++) {
  var imp = IJ.openImage( base_url + names[i] );
  // process image
  // ...
  imp.show();
}

// ERROR: variable 'imp' is not visible outside the loop
// IJ.log("The last image opened was: " + imp);
```

### Arrays

There are both JavaScript arrays and native java arrays.

#### JavaScript arrays

There are many ways to create a JavaScript array. Here are a few:

```java
importClass(Packages.ij.IJ);
// One dimensional:
var names = ["blobs.gif", "boats.gif", "bridge.gif"];
IJ.log("We have " + names.length + " names.");

// Two dimensional:
var coords = [[10, 20, 30],   // X coords
              [15, 25, 35]];  // Y coords

IJ.log( "x0, y0 = " + coords[0][0] + ", " + coords[1][0] );
IJ.log( "All X coords: " + coords[0] );

// Uneven dimensions:
var coords = [[10, 20, 30],               // X coords
              [15, 25, 35, 45, 55, 75]];  // Y coords
IJ.log( "Number of X coords: " + coords[0].length );
IJ.log( "Number of Y coords: " + coords[1].length );
```

Arrays in JavaScript are extremely flexible:

```java
importClass(Packages.ij.IJ);
// Empty arrays:
var names = new Array();
IJ.log("First name is: " + names[0]); // --> prints "undefined", i.e. null.

// Creating array entries at arbitrary index positions:
names[0] = "Table";
names[5] = "Window";
IJ.log("Number of names: " + names.length); // --> prints 6 ! All other entries are "undefined", null.

// Using the array as a dictionary:
names["one"] = 1;
IJ.log("Number of names: " + names.length); // --> still prints 6! But now the array has a map in it as well.
IJ.log("The 'one' is " + names["one"]);     // --> prints 1

// Array entries can contain anything, including other arrays!
names[3] = new Array();
names[3][0] = "Ok";
names[3][1] = "Good";
names[3][2] = ["Arrays", "are", "very", "flexible"]; // another array!
IJ.log(names);
```

#### Native Java arrays

Native java arrays can be passed to java functions and methods directly. For example, to create an array of pixels:

```java
width = 512
height = 512
pixels = java.lang.reflect.Array.newInstance(java.lang.Byte.class, width * height);
print(pixels);
print(pixels.length);
```

To manipulate such arrays, just do so as if they were JavaScript arrays. Beware, though, that now you are limited to numerical indices only, and within the array size only!

```java
width = 512
height = 512
pixels = java.lang.reflect.Array.newInstance(java.lang.Byte.class, width * height);
print(pixels[10]);
java.util.Arrays.fill(pixels, new java.lang.Byte(0));
print(pixels[10]);
// Subtract 25 to each pixel:
for (var i = 0; i < pixels.length; i++) {
  pixels[i] -= 25;
}
print(pixels[10]);
```

With the nashorn JavaScript engine of Java 8, there is an easier way to create native Java arrays. You can create a constructor function that can be reused:

```java
var ByteArray = Java.type("byte[]");
width = 512
height = 512
var pixels = new ByteArray(width*height);
print(pixels.length);
print(pixels[10]);
// Subtract 25 to each pixel:
for (var i = 0; i < pixels.length; i++) {
  pixels[i] -= 25;
}
print(pixels[10]);
var pixels2 = new ByteArray(10);
print(pixels2.length);
```

Above, beware of all problems derived from manipulating signed `byte[]` arrays, whose values should be made unsigned first, modified, then signed back into the array.

### Functions

Simple example:

```java
importClass(Packages.ij.IJ);

function invertImage(imp) {
  var ip = imp.getProcessor();
  ip.invert();
}

// Obtain the current image:
var imp = IJ.getImage();

// Invoke our function
invertImage(imp);

// Update screen:
imp.updateAndDraw();
```

The number of arguments you invoke a function with is flexible. All arguments with which a function is invoked are collected in a variable named `arguments`.

For example:

```java
importClass(Packages.ij.IJ);

function createImage(width, height) {
  IJ.log( "Number of arguments: " + arguments.length);
  
  // Default image type:
  var type = "8-bit";
  
  // Check if an extra argument for the image type was provided:
  if (arguments.length > 2) {
    // Check that the extra arg makes any sense:
    if ("RGB" == arguments[2]) {
      type = "RGB";
    } else {
      IJ.log("Don't know how to use " + arguments[2]);
      return null;
    }
  } 

  var imp = IJ.createImage("New image", type, width, height, 1);
  return imp; 
}

var imp = createImage(400, 400, "RGB");
imp.show();
```

For a complex example see the example script {% include github org='fiji' repo='fiji' branch='master' path='plugins/Examples/Multithreaded_Image_Processing_in_Javascript.js' label='Multithreaded Image Processing in JavaScript' %}, which, beyond parallelization, illustrates how to pass functions as arguments to other functions, and how to invoke them with variable number of arguments.

#### Functions as Objects

Any number of variables may be created on the fly on the body of a function, on its own `this` self reference pointer.

To create an object in JavaScript, just declare a function first that stores the object's data:

```java
// Use uppercase, by convention

function Data(image, annotation) {
    this.image = image;
    this.annotation = annotation;
}
```

Then just create it:

```java
importClass(Packages.ij.IJ);

function Data(image, annotation) {
    this.image = image;
    this.annotation = annotation;
}

var data = new Data(IJ.getImage(), "The current image");

IJ.log("data contains: " + data.image + "\n" + "with annotation: " + data.annotation);
```

To add methods to manipulate the new Data object, create a function that takes the object as an argument:

```java
importClass(Packages.ij.IJ);
importClass(Packages.ij.gui.GenericDialog);

function Data(image, annotation) {
    this.image = image;
    this.annotation = annotation;
}

var data = new Data(IJ.getImage(), "The current image");

IJ.log("data contains: " + data.image + "\n" + "with annotation: " + data.annotation);

function annotate(data) {
    var gd = new GenericDialog("Annotate");
    gd.addStringField("New annotation:", data.annotation, data.annotation.length);
    gd.showDialog();
    if (gd.wasCanceled()) return;
    // assign new annotation:
    var newAnnotation = gd.getNextString();
    IJ.log("Changing the annotation from \"" + data.annotation + "\" to \"" + newAnnotation + "\"");
    data.annotation = newAnnotation;
}

// Invoke on the existing Data object:
annotate(data);
```

As JavaScript uses prototype-based object orientation, we can transform `annotate` to an object method. By adding a function to the prototype of an object, each instance of the object will own this method.

```java
importClass(Packages.ij.IJ);
importClass(Packages.ij.gui.GenericDialog);
 
function Data(image, annotation) {
    this.image = image;
    this.annotation = annotation;
}
 
var data = new Data(IJ.getImage(), "The current image");
 
IJ.log("data contains: " + data.image + "\n" + "with annotation: " + data.annotation);
 
Data.prototype.annotate = function () {
    var gd = new GenericDialog("Annotate");
    gd.addStringField("New annotation:", this.annotation, this.annotation.length);
    gd.showDialog();
    if (gd.wasCanceled()) return;
    // assign new annotation:
    this.annotation = gd.getNextString();
}
 
// Invoke on the existing Data object:
data.annotate();
```

### Creating import namespaces

With Java 8 the new JavaScript engine nashorn introduces the new `JavaImporter`. Instances of `JavaImporter` can be used by JavaScripts `with` statement. This will limit the scope of the imports to the code enclosed by the curly braces of the `with` statement.

The next code snippet shows how to write the annotation example with using the `JavaImporter`:

```java
function Data(image, annotation) {
    this.image = image;
    this.annotation = annotation;
}

function annotate(data) {
    var importer = new JavaImporter(Packages.ij.gui.GenericDialog, Packages.ij.IJ);
    with (importer) {
        var gd = new GenericDialog("Annotate");
        gd.addStringField("New annotation:", data.annotation, data.annotation.length);
        gd.showDialog();
        if (gd.wasCanceled()) return;
        // assign new annotation:
        var newAnnotation = gd.getNextString();
        IJ.log("Changing the annotation from \"" + data.annotation + "\" to \"" + newAnnotation + "\"");
        data.annotation = newAnnotation;
    }
}

var importerIJ = new JavaImporter(Packages.ij.IJ);
with (importerIJ) {
    var data = new Data(IJ.getImage(), "The current image");
    IJ.log("data contains: " + data.image + "\n" + "with annotation: " + data.annotation);
    // Invoke on the existing Data object:
    annotate(data);
}
```

### Inspecting fields and methods of an object

So you are returned an object for a function and you don't know what is it.

To print its class within the interpreter:

```java
ob = ...
ob.getClass();
```

or to the log window:

```java
ob = ...
IJ.log(ob.getClass());
```

To print the list of methods it has, with their return types and argument types:

```java
ob = ...
m = ob.getClass().getMethods();
for (var i=0; i<m.length; i++) IJ.log(m[i]);
```

### Math

All math functions available:

```java
Math.abs(a)     // the absolute value of a
Math.acos(a)    // arc cosine of a
Math.asin(a)    // arc sine of a
Math.atan(a)    // arc tangent of a
Math.atan2(a,b) // arc tangent of a/b
Math.ceil(a)    // integer closest to a and not less than a
Math.cos(a)     // cosine of a
Math.exp(a)     // exponent of a
Math.floor(a)   // integer closest to and not greater than a
Math.log(a)     // log of a base e
Math.max(a,b)   // the maximum of a and b
Math.min(a,b)   // the minimum of a and b
Math.pow(a,b)   // a to the power b
Math.random()   // pseudorandom number in the range 0 to 1
Math.round(a)   // integer closest to a 
Math.sin(a)     // sine of a
Math.sqrt(a)    // square root of a
Math.tan(a)     // tangent of a
```

Built-in constants:

```java
Math.E
Math.PI
```

Trivial example:

```java
var root = Math.sqrt(12);
IJ.log("The root of 12 is " + root);
```

### Functional programming

Suppose you want to create a new image with the square values of the pixels in another image.

First, we get a `source` image (such as the currently active image):


```java
var source = Packages.ij.IJ.getImage(); // the current image (an ImagePlus)
```

Typically, you would then loop through all pixels and apply the result of their square to the other image:

```java
// Return a new ImageProcessor containing the square of each pixel value in ImageProcessor ip
function square(ip) {
  var ip2 = ip.duplicate().convertToFloat();
  var pix = ip.getPixels();

  for (var i = 0; i < pix.length; i++) {
    pix[i] = Math.pow(pix[i], 2);
  }

  return ip2;
}

var ip2 = square( source.getProcessor() );

// Show the result:
new Packages.ij.ImagePlus("square of " + source.title, ip2).show();
```

But imagine now you want to get an image with the square root instead of the square, or the logarithm. Isn't that the same?

We would have to write similar functions named `sqrt` and `pow3`. And so on.

Instead, we should stop and think: there is a common pattern. All we want to do is to apply a function to each pixel in one image, and set the result into the same pixel in another image. In functional programming this pattern is called a `map` operation. Since JavaScript lets us pass functions as arguments, we can define our own `map` function:

```java
function map(fn, ip) {
  var ip2 = ip.duplicate().convertToFloat();
  var pix = ip2.getPixels();

  for (var i = 0; i < pix.length; i++) {
    pix[i] = fn(pix[i]);
  }

  return ip2;
}
```

Now, equipped with our `map` function, we can apply any mathematical operation we want:

```java
var ip_sqrt = map( Math.sqrt, source.getProcessor() );
var ip_log  = map( Math.log,  source.getProcessor() );
...
```

But wait! We didn't pass any extra argument. How can we do a generic function for `pow` so we can apply a power of 2, or 3, etc?

We can rewrite our `map` function like this:

```java
function map(fn, ip) {
  var ip2 = ip.duplicate().convertToFloat();
  var pix = ip2.getPixels();

  for (var i = 0; i < pix.length; i++) {
    pix[i] = fn(pix[i], arguments[2]);
  }
  
  return ip2;
}
```

... where `arguments[2]` is the argument, if any, present beyond any declared arguments. This works because in javascript, functions can accept a variable number of arguments:

```java
var ip2 = map( Math.pow, source.getProcessor(), 2 );
var ip3 = map( Math.pow, source.getProcessor(), 3 );
```
Our second version of the `map` function, though, is a bit perverted: in standard functional programming techniques, the function given as arguments would be applied to each element at index `i` of every list; i.e. the function would receive as many arguments as lists we give to the `map`. But we don't need that.

So what's the big deal? We have abstracted away a common pattern, looping, but furthermore, we have reduced the complexity of our program. So now, for example, applying an optimization to the `map` function will improve `all` the places in our code that use it!

(The same would be true for adding a debugging message, and what not. Anything you want).

For example, since the processing of each pixel is independent of the others, we could parallelize the processing!

```java
function map(fn, ip) {
  var ip2 = ip.duplicate().convertToFloat();
  var pix = ip2.getPixels();

  var n_threads = java.lang.Runtime.getRuntime().availableProcessors();
  var threads = new Array();

  var ai = new java.util.concurrent.atomic.AtomicInteger(0);
  var width = ip.getWidth();
  var height = ip.getHeight();

  var arg = arguments[2];

  for (var t = 0; t < n_threads; t++) {
    threads[t] = new java.lang.Thread( function() {
        // Process one line at a time:
        for (var line = ai.getAndIncrement(); line < height; line = ai.getAndIncrement()) {
           var offset = line * width;
           for (var i = 0; i < width; i++) {
               // invoke function on each pixel, with the optional extra argument
               pix[offset + i] = fn(pix[offset + i], arg);
           }
        }
    });
    threads[t].start();
  }

  // Wait until all threads finish:
  for (var t = 0; t < n_threads; t++) {
    threads[t].join();
  }
  
  return ip2;
}
```

We would call the now multithreaded `map` function just like before:

```java
var ip_sqrt = map( Math.sqrt, source.getProcessor() );
var ip_log  = map( Math.log, source.getProcessor() );
var ip2     = map( Math.pow, source.getProcessor(), 2 );
var ip3     = map( Math.pow, source.getProcessor(), 3 );
```

Above: **beware** that parallelizing a trivial function like Math.sqrt will likely result in a **slower** execution, because of multithreading overheads and the extreme contention in accessing the same pixel array from multiple threads. Perhaps you want to have **two versions** of map: the simple and the parallel one, and use the latter for complex, heavy functions.

For further ease, we could create a `show` function that avoids further repetitions:

```java
function mapAndShow(imp, fn) {
  var ip1 = imp.getProcessor();
  // Map the function to each pixel, into a new ImageProcessor:
  var ip2 = map(fn, ip1, arguments[2]); // pass any extra argument as well
  // Fix LUT range for best visualization:
  ip2.findMinAndMax();
  // Open image in a new window:
  new Packages.ij.ImagePlus(fn.name + " of " + imp.title, ip2).show();
}

with (new JavaImporter(Packages.ij.IJ)) {

  mapAndShow( IJ.getImage(), Math.sqrt);

  mapAndShow( IJ.getImage(), Math.pow, 3);
}
```

{% capture built-in-math-functions %}
All the above are examples to give you an idea of what is possible with JavaScript and ImageJ. If you are interested in applying mathematical functions to images, you may be better off using internal ImageJ commands, as listed in the menu {% include bc path="Process | Math" %}:

```java
with (new JavaImporter(Packages.ij.IJ, Packages.ij.ImagePlus)) {

  var imp = IJ.getImage();
  var imp2 = new ImagePlus("copy of " + imp.title, imp.getProcessor().duplicate().convertToFloat());

  // Set each pixel to the square of its value:
  IJ.run( imp2, "Square", "");

  imp2.show();
}
```
{% endcapture %}
{% include notice icon='note' content=built-in-math-functions %}

### Using other *.js* files as libraries

Programming becomes most useful when code is reused. To that end, many programmers will generalize their code into functions that can be called from other scripts as well.

To avoid copy-pasting all the time (which invariably leads to stale and unmaintainable code), a good idea is to store such general functions in separate *.js* files and let the JavaScript interpreter know about them. You can fetch the location of scripts in a system independent way using `IJ.getDirectory("plugins")`:

```java
load(Packages.ij.IJ.getDirectory("plugins") + "JavaScript" + java.io.File.separator + "my-library-of-useful-functions.js");
// now you can use the functions defined in above .js file
```

To keep your global scope clean, it is recommended to use only functions in your libraries. As JavaScript has a function scope, all variables declared inside a function are not visible in the global scope and can't overwrite variables in the script you are working with.

Best practice is to create a single object in each library that uses the same name as the library.

```java
// This is the code of the library SimpleObject.js
var SimpleObject = function (x, y, val) {
  this.x = x;
  this.y = y;
  this.val = val;
}

// Each new object that is created from SimpleObject owns this function
SimpleObject.prototype.print = function () {
  with (new JavaImporter(Packages.ij.IJ)) {
	IJ.log("x: " + this.x);
	IJ.log("y: " + this.y);
	IJ.log("val: " + this.val);
  }
}
```

The following code demonstrates how to use the library, if it's saved as `SimpleObject.js` in {% include bc path="Plugins|JavaScript" %}.

```java
load(Packages.ij.IJ.getDirectory("plugins") + "JavaScript" + java.io.File.separator + "SimpleObject.js");
var obj = new SimpleObject(10, 10, 100);
obj.print();
```

## ImageJ interaction

### Opening and creating ImageJ images

```java
imp = new Opener().openImage("/path/to/image.jpg");
// do some processing
// ...
imp.show();
```

Or easier:

```java
imp = IJ.openImage("/path/to/image.jpg");
imp.show();
```

Also URLs (in this case, we call directly `show()`, not keeping the returned image pointer into any variable):

```java
IJ.openImage("https://imagej.nih.gov/ij/images/blobs.gif").show();
```

Create an image from scratch, including LUT:

```java
// From scratch:
width = 512
height = 512
pixels = new java.lang.reflect.Array.newInstance(java.lang.Byte.TYPE, width * height);
// process the pixels
// ...
// Create LUT:
channel = new java.lang.reflect.Array.newInstance(java.lang.Byte.TYPE, 256);
for (i=0; i<channel.length; i++)
	channel[i] = Integer(i).byteValue();
cm = new LUT(channel, channel, channel);

// Create the image as 8-bit with the LUT we just created:
imp = new ImagePlus("the title", new ByteProcessor(width, height, pixels, cm));
imp.show();
```

A more convenient way to create images, with a default grayscale LUT:

```
imp = ImagePlus("the title", new ByteProcessor(512, 512));
pixels = imp.getProcessor().getPixels();
// do some processing ...
// ...
imp.show();
```

### Using GenericDialog

This example shows how to get 2 of the open images and a checkbox using GenericDialog.

```java
// get 2 images and a checkbox
var createWindow = true;  //default checkbox value
var imp = null; // default chosen images

var data = getTwoImages( createWindow ); // call the function

if (data) {
  // there was data returned
  imp = [data[0], data[1]];
  createWindow = data[2];

  // show the choices in the log window
  IJ.log(imp[0]); // first image
  IJ.log(imp[1]); // second image
  IJ.log(createWindow); // the status of the checkbox
  // the rest of your code goes here
}



function getTwoImages( createWindow ) {
  var wList = WindowManager.getIDList();

  if (null == wList || wList.length<2){ //there are less than 2 images or no images
	IJ.showMessage("Error", "There must be at least two windows open");
	return;
  }
 
  //get all the image titles so they can be shown in the dialog 
  var titles = new Array();

  for (var i=0, k=0; i<wList.length; i++) {
	var limp = WindowManager.getImage(wList[i]);
	if (limp)
	  titles[k++] = limp.getTitle();
  }

  //construct the dialog
  gd = new GenericDialog("Options");
  gd.addMessage("Binary Reconstruction v 2");
  gd.addChoice("mask i1:", titles, titles[0]);
  gd.addChoice("seed i2:", titles, titles[1]);
  gd.addCheckbox("Create New Window", createWindow);
  gd.showDialog(); //show it

  if (gd.wasCanceled())
	return;

  var i1Index = gd.getNextChoiceIndex();
  var i2Index = gd.getNextChoiceIndex();
  var create_window = gd.getNextBoolean();

  return [WindowManager.getImage(wList[i1Index]),
		  WindowManager.getImage(wList[i2Index]),
		  create_window];
}
```
### Running ImageJ commands

To launch a command:

```java
// in a separate thread:
IJ.doCommand("FFT");

// instead, waiting until it finishes:
IJ.run("FFT");
```

To run commands on a specific image:

```java
// Obtain an image to work on:
var imp = IJ.getImage();

// Call the command to add noise to the current image
// which is here the one provided as argument:
IJ.run(imp, "Add Noise", "");

// Subtract 25 to each pixel value:
IJ.run(imp, "Subtract...", "value=25");
```

Above, you can see what arguments to add to a command by opening the Plugins - Macro Recorder, and then running the command manually. The macro string will print on the recorder window.

### Inspect java methods and fields in an object

To print the static fields and methods of ImagePlus class:

```java
s = "";
for (a in ImagePlus) { s += " " + a; }
```

... which prints `INTEGRATED_DENSITY` `AREA_FRACTION` etc.

To print the fields and methods of an instance of ImagePlus (i.e. an image that already exists):

```java
// get the current image
imp = IJ.getImage();
// print fields and methods
s = ""; for (a in imp) { s += " " + a; }
```

... which prints all method names such as getStatistics isHyperStack etc. and fields like width and height (because there are public 'get' methods for it such as getWidth() and getHeight() .)

### Creating a script for ImageJ

Save your javascript script in a text file:

1.  with extension `.js`
2.  and with an underscore `_` in its name: `my_first_script.js`

...and just drop it in ImageJ's plugins folder or a subfolder.

On startup, the script will appear in the corresponding menu.

If you add the script after ImageJ was started, just call "Help - Update menus" and it will be picked up.

You can continue modifying and saving the script file. Every time you run it from the menu, it will be read from the file system.

## Interfaces and anonymous classes

To create an ImageListener without declaring a new class that implements such java interface, simply use a function that will be mapped to all its methods (as long as they have the same signature, which they do in this case):

```java
ImagePlus.addImageListener( function (imp, name) {
	   if (name == "imageOpened") {
			  IJ.log("Opened image: " + imp);
	  } else if (name == "imageClosed") {
			  IJ.log("Closed image: " + imp);
	  } else if (name == "imageUpdated") {
			  IJ.log("Updated image: " + imp);
	  }
});
```

Alternatively, one can create an object with declared functions inside to assign then to an anonymous class created on the fly from an interface:

```javascript
body = {
  run: function () {
	IJ.log("Running!");
  }
}
// Runnable is an interface
runnable = new Runnable(body);
new Thread(runnable).start();
```

Simplified:
```java
new Thread( function () { IJ.log("Running!"); } ).start();
```

What the code above did: to look for an interface that could take a method with no arguments, represented by the function, and instantiate an anonymous class that implements such interface with the function mapped to its method.

See also an [example plugin](/scripting/comparisons#in-javascript) for ImageJ written in javascript.

## Multithreaded Image Processing in JavaScript

The following example shows how to create a generic function, named `multithreader`, that accepts another function as argument and executes it in parallel a number of times. As a very simple example, a `printer` function is passed to the `multithreader`, and a list of numbers is printed without repeating anyone, and not preserving the order of course.

The `multithreader` scales up to as many CPU cores as the computer has to offer.

Always remember: only **completely** independent tasks can be parallelized effectively!

A good strategy for multithreading involves carefully considering the task to parallelize: how small can the chunks be? For an image, a chunk could be a pixel or a line, but often those are too small to overcome the overhead of parallelization.

Despite the simple example below, the `multithreader` framework function allows variable amount of arguments to be passed, as illustrated in the complete plugin {% include github org='fiji' repo='fiji' branch='master' path='plugins/Examples/Multithreaded_Image_Processing_in_Javascript.js' label='Multithreaded\_Image\_Processing\_in\_Javascript.js' %}. The script shows how to generate an image with random pixel values in a multithreaded manner, and how the choice of chunks to process in parallel is made for reasonable effectiveness.

```java
// Import all classes that are used more than once:
importClass(Packages.ij.IJ);
importClass(Packages.java.lang.Thread);

function multithreader(fun, start, end) {
		var threads = java.lang.reflect.Array.newInstance(Thread.class, java.lang.Runtime.getRuntime().availableProcessors());
		var ai = new java.util.concurrent.atomic.AtomicInteger(start);
		// Prepare arguments: all other arguments passed to this function
		// beyond the mandatory arguments fun, start and end:
		var args = new Array();
		var b = 0;
		IJ.log("Multithreading function \"" + fun.name + "\" with arguments:\n  argument 0 is index from " + start + " to " + end);
		for (var a = 3; a < arguments.length; a++) {
				args[b] = arguments[a];
				IJ.log("  argument " + (b+1) + " is " + args[b]);
				b++;
		}
		var body = {
				run: function() {
						for (var i = ai.getAndIncrement(); i <= end; i = ai.getAndIncrement()) {
								// Execute the function given as argument,
								// passing to it all optional arguments:
								fun(i, args);
								Thread.sleep(100); // NOT NEEDED, just to pretend we are doing something!
						}
				}
		}
		// start all threads
		for (var i = 0; i < threads.length; i++) {
				threads[i] = new Thread(new java.lang.Runnable(body)); // automatically as Runnable
				threads[i].start();
		}
		// wait until all threads finish
		for (var i = 0; i < threads.length; i++) {
				threads[i].join();
		}
}

// The actual desired effect: the printer
function printer(i) {
		IJ.log("i is " + i);
}
 
 // Execute:
 multithreader(printer, 0, 10);
```

See the complete file here: {% include github org='fiji' repo='fiji' branch='master' path='plugins/Examples/Multithreaded_Image_Processing_in_Javascript.js' label='Multithreaded\_Image\_Processing\_in\_Javascript.js' %}

# Links

-   [Tutorial](http://www.mozilla.org/rhino/tutorial.html) at Mozilla Rhino webpages (Java 6).
-   [Scripting Java with JavaScript](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/Rhino/Scripting_Java) (Java 6).
-   [Performance tips](http://www.mozilla.org/rhino/perf.html) (Java 6).
-   [Tutorial](http://winterbe.com/posts/2014/04/05/java8-nashorn-tutorial/) on Oracle Nashorn (Java 8).

See also the [Scripting comparisons](/scripting/comparisons).
