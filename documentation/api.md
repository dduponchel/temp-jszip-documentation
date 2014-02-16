---
title: "JSZip API"
layout: default
---

Documentation
=============

HERE : TOC

JSZip
-----

<a name="constructor"></a>
### new JSZip() or JSZip()

__Description__

Create a new JSZip instance.

__Arguments__

None

__Returns__

A new JSZip.

__Throws__

Nothing.

__Complexity__

Object creation in **O(1)**.

__Example__

```javascript
var zip = new JSZip();
// same as
var zip = JSZip();
```

----------------------------------------

<a name="constructor_load"></a>
### new JSZip(data [,options])

This is a shortcut for

```js
var zip = new JSZip();
zip.load(data, options);
```

Please see the documentation of [load](#load).

----------------------------------------

<a name="file_name"></a>
### file(name)

__Description__

Get a file with the specified name. You can specify folders in the name : the folder separator is a forward slash ("/").

__Arguments__

* name - *string*, the name of the file.

__Returns__

The file if any, ```null``` otherwise. The file has the following structure :
TODO : move to a separate section.

__Throws__

Nothing.

__Complexity__

This is a simple lookup in **O(1)**.

__Examples__

```js
var zip = new JSZip();
zip.file("file.txt", "content");

zip.file("file.txt").name // "file.txt"
zip.file("file.txt").asText() // "content"
zip.file("file.txt").options.dir // false

// utf8 example
var zip = new JSZip(zipFromAjaxWithUTF8);
zip.file("amount.txt").asText() // "&euro;15"
zip.file("amount.txt").asArrayBuffer() // an ArrayBuffer containing &euro;15
zip.file("amount.txt").asUint8Array() // an Uint8Array containing &euro;15

// with folders
zip.folder("sub").file("file.txt", "content");
zip.file("sub/file.txt"); // the file
// or
zip.folder("sub").file("file.txt") // the file
```

---------------------------------------

<a name="file_regex"></a>
### file(regex)

__Description__

Search a file in the current folder and subfolders with a [regular expression](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions). The regex is tested against the relative filename.

__Arguments__

* regex - *RegExp*, the regex to use.

__Returns__

An array of matching files (an empty array if none matched). Each maching file is an instance of [ZipObject](#ZipObject).
TODO : can the user update the data ?

__Throws__

Nothing.

__Complexity__
**O(k)** where k is the number of entries in the current JSZip instance.

__Example__

```js
var zip = new JSZip();
zip.file("file1.txt", "content");
zip.file("file2.txt", "content");

zip.file(/file/); // array of size 2

// example with a relative path :
var folder = zip.folder("sub");
folder
  .file("file3.txt", "content")  // relative path from folder : file3.txt
  .file("file4.txt", "content"); // relative path from folder : file4.txt

folder.file(/file/);  // array of size 2
folder.file(/^file/); // array of size 2, the relative paths start with file

// arrays contain objects in the form:
// {name: "file2.txt", dir: false, asText : function () {...}, ...}
```

---------------------------------------

<a name="file_data"></a>
### file(name, data [,options])

__Description__

Add a file to the zip file.

__Arguments__

* ```name``` - *string*,  the name of the file. You can specify folders in the name : the folder separator is a forward slash ("/").
* ```data```- *String/ArrayBuffer/Uint8Array/Buffer*, the content of the file.
* ```options``` - *object*, the options.
  * ```base64``` - *boolean*, default : ```false```, set to ```true``` if the data is base64 encoded. For example image data from a ```<canvas>``` element. Plain text and HTML do not need this option.
  * ```binary``` - *boolean*, default : ```false```, set to ```true``` if the data should be treated as raw content. If base64 is used, this defaults to ```true```, if the data is not a string, this will be set to true.
  * ```date``` - *date*, default : the current date, the last modification date.
  * ```compression``` - *string*, default : null. If set, specifies compression method to use for this specific file. If not, the default file compression will be used, cf <a href="#doc_generate_options">generate(options)</a>.
  * ```optimizedBinaryString``` - *boolean*, default : false. Set it to true if (and only if) the input is a "binary string" and has already been prepared with a 0xFF mask.

__Returns__

The current JSZip object, for chaining.

__Throws__

An exception if the data is not in a supported format.

__Complexity__

**O(1)** for anything but binary strings.
For binary strings (```data``` is a string and ```binary``` = true), if ```optimizedBinaryString``` is not set, the 0xFF mask will be applied giving a complexity in **O(n)** where n is the size of the added data.

__Example__

```js
zip.file("Hello.txt", "Hello World\n");
zip.file("smile.gif", "R0lGODdhBQAFAIACAAAAAP/eACwAAAAABQAFAAACCIwPkWerClIBADs=", {base64: true});
zip.file("magic.txt", "U2VjcmV0IGNvZGU=", {base64: true, binary: false});
zip.file("Xmas.txt", "Ho ho ho !", {date : new Date("December 25, 2007 00:00:01")});
zip.file("folder/file.txt", "file in folder");

zip.file("animals.txt", "dog,platypus\n").file("people.txt", "james,sebastian\n");

// result : Hello.txt, smile.gif, magic.txt, Xmas.txt, animals.txt, people.txt,
// folder/, folder/file.txt
```

---------------------------------------

<a name="folder_data"></a>
### folder(name)

__Description__

Add a directory to the zip file.

__Arguments__

 * ```name``` - *string*, the name of the directory.

__Returns__

A new JSZip (for chaining), with the new folder as root.

__Throws__

Nothing.

__Complexity__

**O(1)**

__Example__

```js
zip.folder("images");
zip.folder("css").file("style.css", "body {background: #FF0000}");
// or specify an absolute path (using forward slashes)
zip.file("css/font.css", "body {font-family: sans-serif}")

// result : images/, css/, css/style.css, css/font.css
```

---------------------------------------

<a name="folder_regex"></a>
### folder(regex)

__Description__

Search a subdirectory in the current directory with a <a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions">regular expression</a>. The regex is tested against the relative path.

__Arguments__

* regex - *RegExp*, the regex to use.

__Returns__

An array of matching folders (an empty array if none matched). Each maching folder is an instance of [ZipObject](#ZipObject).
TODO : can the user update the data ?

__Throws__

Nothing.

__Complexity__

**O(k)** where k is the number of entries in the current JSZip instance.

__Example__

```js
var zip = new JSZip();
zip.folder("home/Pierre/videos");
zip.folder("home/Pierre/photos");
zip.folder("home/Jean/videos");
zip.folder("home/Jean/photos");

zip.folder(/videos/); // array of size 2

zip.folder("home/Jean").folder(/^vid/); // array of 1
```

---------------------------------------

<a name="filter"></a>
### filter(predicate)

__Description__

Filter nested files/folders with the specified function.

__Arguments__

* ```predicate``` - *function*, the predicate to use.

The predicate has the following signature : ```function (relativePath, file) {...}``` :

* ```relativePath``` - *string*, The filename and its path, reliatively to the current folder.
* ```file``` - *ZipObject*, the file being tested. See [ZipObject](#ZipObject).

The predicate must return true if the file should be included, false otherwise.


__Returns__

An array of matching ZipObject.

__Throws__

Nothing.

__Complexity__

**O(k)** where k is the number of entries.

__Example__

```js
var zip = new JSZip().folder("dir");
zip.file("readme.txt", "content");
zip.filter(function (relativePath, file){
  // relativePath == "readme.txt"
  // file = {name:"dir/readme.txt",options:{...},asText:function}
  return true/false;
});
```

---------------------------------------

<a name="remove"></a>
### remove(name)

__Description__

Delete a file or folder (recursively).

__Arguments__

* ```name``` - *string*, the name of the file/folder to delete.

__Returns__

The current JSZip object.

__Throws__

Nothing.

__Complexity__

**O(k)** where k is the number of entry to delete (may be > 1 when removing a folder).

__Example__

```js
var zip = new JSZip();
zip.file("Hello.txt", "Hello World\n");
zip.file("temp.txt", "nothing").remove("temp.txt");
// result : Hello.txt

zip.folder("css").file("style.css", "body {background: #FF0000}");
zip.remove("css");
//result : empty zip
```

---------------------------------------

<a name="generate"></a>
### generate(options)

__Description__

Generates the complete zip file.

__Arguments__

* ```options``` - *object*, the options to generate the zip file :
  * ```base64``` - *boolean*, **deprecated**, default : ```false```, use ```type``` instead. If ```type``` is not used, set to ```false``` to get the result as a raw byte string, ```true``` to encode it as base64.
  * ```compression``` - *string*, default ```STORE``` (no compression), the default file compression method to use. Available methods are ```STORE``` and ```DEFLATE```. You can also provide your own compression method.
  * ```type``` - *string*, default : ```base64```. The type of zip to return, see below for the other types.

Possible values for ```type``` :

* ```base64``` (default) : the result will be a string, the binary in a base64 form.
* ```string``` : the result will be a string in "binary" form, saving 1 byte per char (2 bytes).
* ```uint8array``` : the result will be a Uint8Array containing the zip. This requires a compatible browser.
* ```arraybuffer``` : the result will be a ArrayBuffer containing the zip. This requires a compatible browser.
* ```blob``` : the result will be a Blob containing the zip. This requires a compatible browser.
* ```nodebuffer``` : the result will be a nodejs Buffer containing the zip. This requires nodejs.

HTML5 note : when using type = "uint8array", "arraybuffer" or "blob", be sure to check if the browser supports it (you can use <a href="#jszip_support"><code>JSZip.support</code></a>).

__Returns__

The generated zip file.

__Throws__

An exception if the asked ```type``` is not available in the browser, see [JSZip.support](#support).

__Complexity__

TODO : worst case, with/out compression, etc

__Example__

```js
var content = zip.generate(); // base64
location.href="data:application/zip;base64,"+content;
```

```js
var blobLink = document.getElementById('blobLink');
blobLink.download = "hello.zip";
blobLink.href = window.URL.createObjectURL(
   zip.generate({type:"blob"})
);
```

```js
var content = zip.generate({type:"nodebuffer"});
require("fs").writeFile("hello.zip", content, function(err){/*...*/});
```

---------------------------------------

<a name="load"></a>
### load(data [, options])

__Description__

Read an existing zip and merge the data in the current JSZip object. This technique has some limitations, see <a href="#zip_load_limits">below</a>.

__Arguments__

* ```data``` - *String/ArrayBuffer/Uint8Array/Buffer*, the zip file
* ```options``` - *object*, the options to load the zip file :
  * ```base64``` - *boolean*, default : false, set to ```true``` if the data is base64 encoded, <code>false</code> for binary.
  * ```checkCRC32``` - *boolean*, default : false, set to ```true``` if the read data should be checked against its CRC32. Please see the "complexity" paragraph before using this option.
  * ```optimizedBinaryString``` - *boolean*, default : false, set to true if (and only if) the input is a string and has already been prepared with a 0xFF mask.


Zip features supported by this method :

* Compression (<code>DEFLATE</code> supported)
* zip with data descriptor
* ZIP64
* UTF8 in file name, UTF8 in file content

Zip features not (yet) supported :

* password protected zip
* multi-volume zip

__Returns__

The current JSZip object.

__Throws__

An exception if the loaded data is not valid zip data or if it uses features (multi volume, password protected, etc).

__Complexity__

For k the number of entries in the zip file and n the length of the data :

The default use case is **O(k)**.
If the data is in base64, we must first decode it : **O(k + n)**.
If the data is a string not in base64 and optimizedBinaryString is false, we must apply the 0xFF mask : **O(k + n)**.
If checkCRC32 is true, it **adds** to the above complexity **O(n)** and the complexity of the decompression algorithm.

__Example__

```js
var zip = new JSZip();
zip.load(zipDataFromXHR);
```

```js
require("fs").readFile("hello.zip", function (err, data) {
  if (err) throw err;
  var zip = new JSZip();
  zip.load(data);
}
```

---------------------------------------

<a name="support"></a>
JSZip.support
-------------

If the browser supports them, JSZip can take advantage of some "new" features : ArrayBuffer, Blob, Uint8Array.
To know if JSZip can use them, you can check the JSZip.support object. It contains the following boolean properties :

* ```arraybuffer``` : true if JSZip can read and generate ArrayBuffer, false otherwise.
* ```uint8array``` : true if JSZip can read and generate Uint8Array, false otherwise.
* ```blob``` : true if JSZip can read and generate Blob, false otherwise.
* ```nodebuffer``` : true if JSZip can read and generate nodejs Buffer, false otherwise.

---------------------------------------

<a name="zipobject"></a>
ZipObject
---------

This represents an entry in the zip file. If the entry comes from an existing archive previously [loaded](#load), the content may need to be decompressed/converted first.

__Attributes__

* ```name```  - *string*, the absolute path of the file
* ```options``` - *object*, the options of the file. The available options are :
  * ```base64``` - *boolean*, cf <a href="#doc_file_name_data_options">file(name, data [,options])</a>
  *  ```binary``` - *boolean*, cf <a href="#doc_file_name_data_options">file(name, data [,options])</a>
  * ```dir``` - *boolean*, true if this is a directory
  * ```date``` - *date*, cf <a href="#doc_file_name_data_options">file(name, data [,options])</a>
  * ```compression``` - *compression*, cf <a href="#doc_file_name_data_options">file(name, data [,options])</a>

__Methods__


### asText()

__Description__


TODO

<li><code>asText()</code>, string, the content as an utf8 string (utf8 encoded if necessary).</li>
<li><code>asBinary()</code>, string, the content as binary (utf8 decoded if necessary).</li>
<li><code>asArrayBuffer()</code>, ArrayBuffer, need a <a href="#jszip_support">compatible browser</a>.</li>
<li><code>asUint8Array()</code>, Uint8Array, need a <a href="#jszip_support">compatible browser</a>.</li>
<li><code>asNodeBuffer()</code>, nodejs Buffer, need <a href="#jszip_support">nodejs</a>.</li>
