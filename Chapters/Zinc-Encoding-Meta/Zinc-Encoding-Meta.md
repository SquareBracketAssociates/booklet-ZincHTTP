## Character Encoding and Resource Meta Description

@cha:zincEncoding


The rise of the Internet and of Open Standards resulted in the adoption of a number of fundamental mechanisms to enable communication and collaboration between
different systems.

One such mechanism is the ability to encode strings or characters to bytes or to decode strings or characters from bytes. Different encoding standards have been
developed over the years and Pharo supports many current and legacy encodings.

Another important aspect of collaboration is the ability to describe resources such as files. Both Mime-Type and URLs or URIs are basic building blocks for creating meta
descriptions of resources and Pharo also has objects that implement these fundamental aspects.

In this chapter we discuss Character encoding, MIME types and URL/URIs. They are essential for the correct implementation of HTTP, but they are independent from it, as they are used for many other purposes.


### Character Encoding


We will first show how to get Unicode from characters and strings within Pharo.
We will then show how to decode and encode characters and strings from and to bytes.


### Characters and Strings use Unicode Internally


Proper character encoding and decoding is crucial in today's international world. Internally, Pharo stores characters and strings using Unicode.
[Unicode](http://en.wikipedia.org/wiki/Unicode) is a very large internationally standardized collection of code points \(integer numbers\) representing all of the world
languages' characters.

We can obtain the code point \(Unicode value\) of a character by sending it the `codePoint` message, for example:

```
$H codePoint
>>> 72
```


Here are some example strings in multiple languages with their Unicode code points:

```
'Hello' collect: #codePoint as: Array.
>>> #(72 101 108 108 111)
```


```
'Les élèves français' collect: #codePoint as: Array.
>>> #(76 101 115 32 233 108 232 118 101 115
        32 102 114 97 110 231 97 105 115)
```


```
'Ελλάδα' collect: #codePoint as: Array.
>>> #(917 955 955 940 948 945)
```


For a simple language like English, all characters have code points below 128 \(which fits in 7 bits, for historical reasons\).
These characters are part of [ASCII](http://en.wikipedia.org/wiki/ASCII). The very first part of the so called Basic Multilingual Plane of Unicode \(the first 128 code points of it\) are identical to ASCII.

```
$a codePoint
>>> 97
```


Next come a number of European languages, like French, which have code points below 256 \(fitting in 8 bits or one byte\).
These characters are part of [Latin-1 \(ISO-8859-1\)](http://en.wikipedia.org/wiki/ISO/IEC_8859-1), whose first 256 code points are identical in Unicode.

```
$é codePoint
>>> 233
```


And finally, there are hundreds of other languages, like Chinese, Japanese, Cyrillic, Arabic or Greek. You can see from the example above: Greece written in Greek, that those code points are higher than 256 \(and thus no longer fit in one byte\).

```
$λ codePoint
>>> 955
```


Unicode code points are often written using a specific hexadecimal notation. For example, the previous character, the Greek lowercase lambda, is written as
`U+03BB`. The Pharo inspector also shows this value next to the codepoint.

The good thing is, we can work with text in any language in Pharo. However, to display everything correctly a font must be used that is capable of showing all the
characters \(or glyphs\) needed, for example Arial Unicode MS.


### Encoding and Decoding


For communication with the world outside Pharo, the operating system, files, the internet, et cetera, we have to represent our strings as a collection of bytes.
Yet code points are different to bytes, as will be shown below. Therefore we need a way to transform our internal strings into external collection of bytes and vice versa.

Character encoding is the standard way of converting a native Pharo string, i.e. a collection of Unicode code points, to a series of bytes. Character decoding is the reverse
process: interpreting a series of bytes as a collection of Unicode code points, to create a Pharo string.

To implement character encoding or decoding, a concrete subclass of
the class `ZnCharacterEncoder` is used, e.g.  `ZnUTF8Encoder`. Character encoders do the following:

- encode a character \(message `nextPut:toStream:`\) or string \(message `next:putAll:startingAt:toStream:`\) onto a binary stream
- convert a string \(`encodeString:`\) to a byte array
- decode a binary stream to a character \(`nextFromStream:`\) or string \(`readInto:startingAt:count:fromStream:`\)
- convert a byte array to string \(`decodeBytes:`\)
- compute the number of bytes that are needed to encode a character \(`encodedByteCountFor:`\) or string \(`encodedByteCountForString:`\)
- move a binary stream backwards one character \(`backOnStream:`\)


Character encoders do proper error handling, throwing an error of the
class `ZnCharacterEncodingError` when something goes wrong. The strict/lenient setting controls some behavior in this respect, and this will be discussed later in this chapter.

The recommended encoding is the primary internet encoding: [UTF-8](http://en.wikipedia.org/wiki/UTF-8).  It is a variable length encoding that is
optimized somewhat for ASCII and to a lesser degree for Latin1 and some other common European encodings.


### Converting Strings and ByteArrays


The first use of encoders is to convert Strings to ByteArrays and vice-versa.
We however deal only indirectly with character encoders. The `ByteArray` and  `String` classes have some convenience methods to do encoding and decoding:

```
'Hello' utf8Encoded.
>>> #[72 101 108 108 111]

'Hello' encodeWith: #latin1.
>>> #[72 101 108 108 111]
```


Our ASCII string, `'Hello'` encodes identically using either UTF-8 or Latin-1.

```
'Les élèves français' utf8Encoded.
>>> #[76 101 115 32 195 169 108 195 168 118 101 115
        32 102 114 97 110 195 167 97 105 115]

'Les élèves français' encodeWith: #latin1.
>>> #[76 101 115 32 233 108 232 118 101 115
       32 102 114 97 110 231 97 105 115]
```


Our French string, `'Les élèves français'`, encodes differently though.
The reason is that UTF-8 uses two bytes for the accented letters like é, è and ç.
Note how for Latin-1, and **only** for Latin-1 and ASCII, the Unicode code points are equal to the encoded byte values.

```
'éèç' utf8Encoded.
>>> #[195 169 195 168 195 167]

'éèç' encodeWith: #latin1.
>>> #[233 232 231]

'éèç' collect: #codePoint as: ByteArray
>>> #[233 232 231]
```


```
'Ελλάδα' utf8Encoded.
>>> #[206 149 206 187 206 187 206 172 206 180 206 177]

'Ελλάδα' encodeWith: #latin1.
>>> ZnCharacterEncodingError: 'Character Unicode code point outside encoder range'
```


Our greek string, `'Ελλάδα'`, gives an error when we try to encode it using Latin-1.
The reason is that the Greek letters are outside of the alphabet of Latin-1. Still, UTF-8 manages to encode them using just two bytes.

The reverse process, decoding, is equally simple:

```
#[72 101 108 108 111] utf8Decoded.
>>> 'Hello'

#[72 101 108 108 111] decodeWith: #latin1.
>>> 'Hello'
```


```
#[76 101 115 32 195 169 108 195 168 118 101 115
  32 102 114 97 110 195 167 97 105 115] utf8Decoded.
>>> 'Les élèves français'

#[76 101 115 32 195 169 108 195 168 118 101 115
  32 102 114 97 110 195 167 97 105 115] decodeWith: #latin1.
>>> 'Les Ã©lÃ¨ves franÃ§ais'

#[76 101 115 32 233 108 232 118 101 115
  32 102 114 97 110 231 97 105 115] utf8Decoded.
>>> ZnInvalidUTF8: 'Illegal continuation byte for utf-8 encoding'
```

```
#[76 101 115 32 233 108 232 118 101 115
  32 102 114 97 110 231 97 105 115] decodeWith: #latin1.
>>> 'Les élèves français'

#[206 149 206 187 206 187 206 172 206 180 206 177] utf8Decoded.
>>> 'Ελλάδα'

#[206 149 206 187 206 187 206 172 206 180 206 177] decodeWith: #latin1.
>>> ZnCharacterEncodingError: 'Character Unicode code point outside encoder range'
```


Our English `'Hello'`, being pure ASCII, can be decoded using either UTF-8 or Latin-1. Our French `'Les élèves français'` is another story: using the wrong
encoding gives either gibberish or `ZnInvalidUTF8` error. The same is true for our Greek `'Ελλάδα'`.

You might wonder why in the first case the `latin1` encoder produced gibberish, while in the second case it gave an error.
This is because in the second case, there was a byte with value 149, which is outside its alphabet. So called byte encoders, like Latin-1,
take a subset of Unicode characters and compress them in 256 possible byte values. This can be seen by inspecting the character or byte domains of a `ZnByteEncoder`, as follows:

```
(ZnByteEncoder newForEncoding: 'iso-8859-1') byteDomain.
(ZnByteEncoder newForEncoding: 'ISO_8859_7') characterDomain.
```


Note that identifiers for encodings are interpreted flexibly \(case and punctuation do not matter\).

There exists a special `ZnNullEncoder` that basically does nothing: it treats bytes are characters and vice versa. This is actually mostly equivalent to Latin-1 or ISO-8859-1. \(And yes, that is a bit confusing.\)


### Converting Streams



The second primary use of encoders is when dealing with streams. More specifically, when interpreting a binary read or write stream as a character stream.
Note that at their lowest level, all streams to and from the operating system or network are binary and thus need the use of an encoder when treating them as
character streams.

To treat a binary write stream as a character write stream, wrap it
with a `ZnCharacterWriteStream`. Similary, `ZnCharacterReadStream` should be used to treat a binary read stream as a
character stream. Here is an example:

```
'encoding-test.txt' asFileReference writeStreamDo: [ :out |
   (ZnCharacterWriteStream on: out binary encoding: #utf8)
      nextPutAll: 'Hello'; space; nextPutAll: 'Ελλάδα'; crlf;
      nextPutAll: 'Les élèves français'; crlf ].

'encoding-test.txt' asFileReference readStreamDo: [ :in |
   (ZnCharacterReadStream on: in binary encoding: #utf8)
      upToEnd ]

>>> 'Hello Ελλάδα
Les élèves français
'
```


We used the message `on:encoding:` here, but there is also a plain message `on:` instance creation message that defaults to the UTF-8 encoding.
Internally, the character streams will use an encoder instance to do the actual work.


### ByteStrings and WideStrings are Concrete Subclasses of String



Up until now we spoke about Strings as being a collection of Characters, each of which is represented as a Unicode code point.
And this is conceptually totally how they should be thought about. However, in reality, the class `String` is an abstract class with two concrete subclasses.
This will show up when inspecting `String` instances, so it is important
to understand what is going on. Consider the following example strings:

```
'Hello' class
>>> ByteString
```


```
'Les élèves français' class
>>> ByteString
```


```
'Ελλάδα' class
>>> WideString
```


Simple ASCII strings are ByteStrings. Strings using special characters may be WideStrings or may still be ByteStrings.
The explanation of the use of the `WideString` or `ByteString` class is very simple when considering the Unicode code points used for each character.

In the first case, for ASCII, the code points are always less than 128. Hence they fit in one byte. The second string is using Latin-1 characters, whose code
points are less than 256. These still fit in a byte. A `ByteString` is a `String` that
only stores Unicode code points that fit in a byte, in an implementation that is very efficient. Note that `ByteString` is a variable byte subclass of `String`.

Our last example has code points that no longer fit in a byte. To be able to store these, `WideString` allocates 32-bit \(4 byte\) slots for each character. This
implementation is necessarily less efficient. Note that `WideString` is a variable word subclass of `String`.

In practice, the difference between `ByteString` and `WideString` should not matter. Conversions are done automatically when needed.

```
'abc' copy at: 1 put: $α; class.
>>> WideString
```


As the above example shows, in a `ByteString` `'abc'` putting the Unicode character `$α`, converts it to a `WideString`. \(This is actually done using a `becomeForward:` message.\) When benchmarking, this conversion might show up as taking significant time. If you know upfront that you will need WideStrings, it can be better to start with the right type.


### ByteString and ByteArray Equivalence is an Implementation Detail


There is another implementation detail worth mentioning: for the Pharo virtual machine, more specifically, for a number of primitives, `ByteString` and `ByteArray` instances are equivalent. Given what we now know, that makes sense. Consider the following code:

```
'abcdef' asByteArray.
>>> #[97 98 99 100 101 102]

'ABC' asByteArray.
>>> #[65 66 67]

'abcdef' copy replaceFrom: 1 to: 3 with: #[65 66 67].
>>> 'ABCdef'

#[97 98 99 100 101 102] copy replaceFrom: 1 to: 3 with: 'ABC'.
>>> #[65 66 67 100 101 102]
```


In the third expression, we send the message `replaceFrom:to:with:` on a `ByteString`, but give a `ByteArray` as third argument. So we are replacing part of a `ByteString` with a `ByteArray`. And it works!

The last example goes the other way around: we replace part of a `ByteArray` with a `ByteString`, which works as well.

What about doing the same mix up with elements ?

```
'abc' copy at: 1 put: 65; yourself.
>>> Error: improper store into indexable object

#[97 98 99] copy at: 1 put: $A; yourself.
>>> Error: improper store into indexable object
```


This is more what we expect: we're not allowed to do this. We are mixing two types that are not equivalent, like `Character` and `Integer`.

So although it is true that there is some equivalence between `ByteString` and `ByteArray`, you should not mix up the two. 
It is an implementation detail that you should not rely upon.


### Beware of Bogus Conversions


Given a string, it is tempting to send it the message `asByteArray` to convert it to bytes. Similary, it is tempting to convert a byte array by
sending it the message `asString`. These are however bogus conversions that should not be used as for some strings they will work, but for others not. Success depends on the code points of the characters in the string. Basically
the conversion is possible for strings for which the following property holds:

```
'Hello' allSatisfy: [ :each | each codePoint < 256 ].
>>> true

'Les élèves français' allSatisfy: [ :each | each codePoint < 256 ].
>>> true

'Ελλάδα' allSatisfy: [ :each | each codePoint < 256 ].
>>> false
```


Now, even though the first two can be converted, they will not be using the same encoding. Here is a way to explicitly express this idea:

```
#(null ascii latin1 utf8) allSatisfy: [ :each |
  ('Hello' encodeWith: each) = 'Hello' asByteArray ].
>>> true.

('Les élèves français' encodeWith: #latin1) = 'Les élèves français' asByteArray.
>>> true.

('Les élèves français' encodeWith: #null) = 'Les élèves français' asByteArray.
>>> true.

'Les élèves français' utf8Encoded = 'Les élèves français' asByteArray.
>>> false.
```


For pure ASCII strings, with all code points below 128, no encoding \(null encoding\), ASCII, Latin-1 and UTF-8 are all the same. For other `ByteString`
 instances, like `'Les élèves français'`, only Latin-1 works. In that case it is also equivalent of doing no encoding.

The lazy conversion for proper Unicode WideStrings will give unexpected results:

```
'Ελλάδα' asByteArray.
>>> #[0 0 3 149 0 0 3 187 0 0 3 187 0 0 3 172 0 0 3 180 0 0 3 177]
```


This 'conversion' does not correspond to any known encoding. It is the result of writing 4-byte Unicode code points as Integers.

!!note Using this is a bug no matter how you look at it. In this century you will look silly for not implementing proper support for all languages. When converting between strings and bytes, use a proper, explicit encoding.


### Strict and Lenient Encoding


No encoding \(or the null encoder\) and Latin-1 encoding are in fact not completely the same. This is because there are 'holes' in the table: some byte
values are undefined, which a strict encoder won't allow, and the default encoder is strict.

For example, the Unicode code point 150 is strictly speaking not in Latin-1:

```
ZnByteEncoder latin1 encodeString: 150 asCharacter asString.
>>> ZnCharacterEncodingError: 'Character Unicode code point outside encoder range'

ZnByteEncoder latin1 decodeBytes: #[ 150 ].
>>> ZnCharacterEncodingError: 'Character Unicode code point outside encoder range'
```


The encoder can however be instructed to `beLenient`, which will produce a silent conversion \(if that is possible\). In this case, Unicode character 150 \(`U+0096`\) is an unprintable control character meaning 'Start of Protected Area' \(SPA\) and is strictly speaking not part of Latin-1.

```
ZnByteEncoder latin1 beLenient encodeString: 150 asCharacter asString.
>>> #[ 150 ]

ZnByteEncoder latin1 beLenient decodeBytes: #[ 150 ].
>>> ''
```


You can explicity access both the allowed byte or character values, i.e. the domain of encoder or decoder:

```
ZnByteEncoder latin1 characterDomain includes: 150 asCharacter.
>>> false

ZnByteEncoder latin1 byteDomain includes: 150.
>>> false
```


Note that the lower half of a byte encoding, the ASCII part between 0 and 127, is always treated as a one to one mapping.

### Available Encoders


Pharo comes with support for the most important encodings currently used, as well as with support for some important legacy encodings. 
Seen as the classes implementing them, the following encoders are available:
- `ZnUTF8Encoder`
- `ZnUTF16Encoder`
- `ZnByteEncoder`
- `ZnNullEncoder`


Where `ZnByteEncoder` groups a large number of encodings. This list is available as `ZnByteEncoder knownEncodingIdentifiers`. Here is a list of all recognized, canonical names: arabic, cp1250, cp1251, cp1252, cp1253, cp1254, cp1255, cp1256, cp1257, cp1258, cp850, cp866, cp874, cyrillic, dos874, doslatin1, greek, hebrew, ibm819, ibm850, ibm866, iso885910, iso885911, iso885913, iso885914, iso885915, iso885916, iso88592, iso88593, iso88594, iso88595, iso88596, iso88597, iso88598, iso88599, koi8, koi8r, koi8u, latin2, latin3, latin4, latin5, latin6, mac, maccyrillic, macintosh, macroman, oem850, windows1250, windows1251, windows1252, windows1253, windows1254, windows1255, windows1256, windows1257, windows1258, windows874, xcp1250, xcp1251, xcp1252, xcp1253, xcp1254, xcp1255, xcp1256, xcp1257, xcp1258, xmaccyrillic and xmacroman.

### Mime-Types


A mime-type is a standard, cross-platform definition of a file or document type or format. The official term is an [Internet media type](http://en.wikipedia.org/wiki/Internet_media_type).

Mime-types are modeled using `ZnMimeType` objects, which have 3 components:

1. a main type, for example `text` or `image`,
1. a sub type, for example `plain` or `html`, or `jpeg`, `png` or `gif`, and
1. a number of attributes, for example `charset=utf-8`.


The mime-type syntax is as follows:

`<main>/<sub> [;<param1>=<value1>[,<param2>=<value2>]*]`.

#### Creating Mime-Types


Instances of `ZnMimeType` are created by explicitly specifying its components, through parsing a string or by accessing predefined values. In any case, a new instance is always created.

The class side of `ZnMimeType` has some convenience methods \(in the protocol `convenience`\) for accessing well known mime-types, which is the recommended way for obtaining these mime-types:

```
ZnMimeType textHtml
>>> text/plain;charset=utf-8

ZnMimeType imagePng
>>> image/png
```


Here is an example of how to create a mime-type by explicitly specifying its components:

```
ZnMimeType main: 'image' sub: 'png'
>>> image/png
```


The main parsing interface of `ZnMimeType` is the class side `fromString:` message.

```
ZnMimeType fromString: 'image/png'
>>> image/png
```


To make it easier to write code that accepts both instances and strings, the `asZnMimeType` message can be used:

```
'image/png' asZnMimeType
>>> image/png

ZnMimeType imagePng asZnMimeType = 'image/png' asZnMimeType
>>> true
```


Finally, `ZnMimeType` also knows how to convert file name extensions to mime-types using the `forFilenameExtension:` message. This mapping is based on the
Debian/Ubuntu `/etc/mime.types` file, which is encoded into the
method `mimeTypeFilenameExtensionsSpec`.

```
ZnMimeType forFilenameExtension: 'html'.
>>> text/html;charset=utf-8
```


In most applications, the concept of a default mime-type exists. It basically means: we don't know what these bytes represent.

```
ZnMimeType default
>>> application/octet-stream
```



### Working with Mime-Types


Once you have a ZnMimeType instance, you can access its components using the `main`, `sub` and `parameters` messages.

An important aspect of mime-types is whether the type is textual or binary, which is testable with the `isBinary` message. Typically, text, XML or JSON are considered
textual, while images are binary.

For textual \(non-binary\) types, the encoding \(or charset parameter\) defaults to UTF-8, the prevalent internet standard. With the convencience messages
`charSet:`, `setCharSetUTF8` and `clearCharSet` you can manipulate the charset parameter.

Comparing mime-types using the standard `=` message takes all components into account, including the parameters. Different parameters lead to different
mime-types. As a result, when charsets are involved it is often better to compare using the `matches:` message, as follows:

```
'text/plain' asZnMimeType = ZnMimeType textPlain.
>>> false

ZnMimeType textPlain = 'text/plain' asZnMimeType.
>>> false

'text/plain' asZnMimeType matches: ZnMimeType textPlain.
>>> true

ZnMimeType textPlain matches: 'text/plain' asZnMimeType.
>>> true
```


The charset=UTF-8 that is part of what `ZnMimeType textPlain` returns is not taken into account in the second set of comparisons.

The main or sub types can be a wildcard, indicated by a `*`. This allows for matching. Obviously, everything matches `*/*` \(`ZnMimeType any`\). Otherwise,
when the sub type is `*`, the main types must be equal. Here is an example.

```
ZnMimeType text.
>>> text/*

ZnMimeType textHtml matches: ZnMimeType text.
>>> true

ZnMimeType textPlain matches: ZnMimeType text.
>>> true

ZnMimeType applicationXml matches: ZnMimeType text.
>>> false
```



### URLs


URLs \(or URIs\) are a way to name or identify an entity. Often, they also contain information of where the entity they name or identify can be accessed.

We will be using the terms URL \([Uniform Resource Locator](http://en.wikipedia.org/wiki/Uniform_resource_locator)\) and URI
\([Uniform Resource Identifier](http://en.wikipedia.org/wiki/Uniform_resource_identifier)\) interchangeably, as is most commonly done in practice. A URI is just a
name or identification, while a URL also contains information on how
to find or access a resource. Consider the following example: the URI `/documents/cv.html`
identifies and names a document, while the URL `http://john-doe.com/documents/cv.html` also specifies that we can use HTTP to access this
resource on a specific server.

By considering most parts of an URL as optional, we can use one abstraction to implement both URI and URL using one class. The class `ZnUrl` models URLs \(or URIs\) and has the following components:

1. scheme - like `#http`, `#https `, `#ws`, `#wws`, `#file ` or ` nil`
1. host - hostname string or `nil `
1. port - port integer or `nil`
1. segments - collection of path segments, ends with `#/` for directories
1. query - query dictionary or `nil`
1. fragment - fragment string or `nil`
1. username - username string or `nil`
1. password - password string or `nil`


The syntax of the external representation of a ZnUrl informally looks like this:
`scheme://username:password@host:port/segments?query#fragment`

### Creating URLs


ZnUrls are most often created by parsing an external representation using either the `fromString:` class message or by sending the `asUrl` or `asZnUrl`
convenience message to a string.

```
ZnUrl fromString: 'http://www.google.com/search?q=Smalltalk'.
'http://www.google.com/search?q=Smalltalk' asUrl.
```


The same instance can also be constructed programmatically:

```
ZnUrl new
   scheme: #http;
   host: 'www.google.com';
   addPathSegment: 'search';
   queryAt: 'q' put: 'Smalltalk';
   yourself.
```


`ZnUrl` components can be manipulated destructively. Here is an example:

```
'http://www.google.com/?one=1&two=2' asZnUrl
   queryAt: 'three' put: '3';
   queryRemoveKey: 'one';
   yourself.
   
>>> http://www.google.com/?two=2&three=3
```


### External and Internal Representation of URLs


Some characters of parts of a URL are considered as illegal because including them would interfere with the syntax and further processing. They thus have to be encoded. The methods of `ZnUrl` in the
`accessing` protocols do not do any encoding, while those in `parsing` and `printing` do. Here is an example:

```
'http://www.google.com'
   addPathSegment: 'an encoding';
   queryAt: 'and more' put: 'here, too';
   yourself
>>> http://www.google.com/an%20encoding?and%20more=here,%20too
```


The ZnUrl parser is somewhat forgiving and accepts some unencoded URLs as well, like most browsers would.

```
'http://www.example.com:8888/a path?q=a, b, c' asZnUrl.
>>> http://www.example.com:8888/a%20path?q=a,%20b,%20c
```


### Relative URLs


ZnUrl can parse in the context of a default scheme, like a browser would do.

```
ZnUrl fromString: 'www.example.com' defaultScheme: #http
>>> http://www.example.com/
```


Given a known scheme, ZnUrl knows its default port, and this is accessed by `portOrDefault`.

A path defaults to what is commonly referred to as slash, which is testable with `isSlash`. Paths are most often \(but don't have to be\) interpreted as filesystem paths. To support this, the `isFilePath` and `isDirectoryPath` tests and `file` and `directory` accessors are provided.

ZnUrl has some support to handle one URL in the context of another one, this is also known as a relative URL in the context of an absolute URL. This is implemented using the `isAbsolute`, `isRelative` and `inContextOf:` methods. For example:

```
'/folder/file.txt' asZnUrl inContextOf: 'http://fileserver.example.net:4400' asZnUrl.
>>> http://fileserver.example.net:4400/folder/file.txt
```



### Operations on URLs


To add operations to URLs you could add an extension method to the ZnUrl class. In many cases though, it will not work on all kinds of URLs but only on a subset. In other words, you need to dispatch, not just on the scheme but maybe even on other URL elements. That is where `ZnUrlOperation` comes in.

The first step for its use is defining a name for the operation. For example, the symbol `#retrieveContents`. Second, one or more subclasses of
`ZnUrlOperation` need to be defined, each defining the class side message `operation` to return the name, `#retrieveContents` in the example. Then all subclasses with the same operation form the group of applicable implementations. Third, these handler subclasses overwrite `performOperation` to do the actual work.

Given a ZnUrl instance, sending the message `performOperation:` or
`performOperation:with:` will send the message `performOperation:with:on:` to `ZnUrlOperation`. In turn, it
will look for an applicable handler subclass, instanciate and invoke it.

Each subclass will be sent `handlesOperation:with:on:` to test if it can handle the named operation with an optional argument on a specific URL. The default implementation already covers the most common case: the operation name has to match and the scheme of the URL has to be part of the collection returned by `schemes`.

For our example, the message `retrieveContents` on ZnUrl is implemented as an operation named `#retrieveContents`. The handler class is either the class  `ZnHttpRetrieveContents`
for the schemes `http` and `https` or the class `ZnFileRetrieveContents` for the scheme `file`.

This dispatching mechanism is more powerful than scheme specific `ZnUrl` subclasses because other elements can be taken into account. It also addresses another issue with scheme specific `ZnUrl` subclasses, which is that there are an infinite number of schemes which no hierarchy could cover.

### Odds and Ends


Sometimes, the combination of a host and port are referred to as authority, and this is accessable with the `authority` message.

There are convenience methods to download the resource a ZnUrl points
to: `retrieveContents`  and `saveContentsToFile`. The first retrieves the contents and returns it directly, while the expression saves the contents directly to a file.

```
'http://zn.stfx.eu/zn/numbers.txt' asZnUrl retrieveContents.
'http://zn.stfx.eu/zn/numbers.txt' asZnUrl saveContentsToFile: 'numbers.txt'.
```


ZnUrl can be used to handle file URLs. Use `isFile` to test for this scheme.

Given a file URL, it can be converted to a regular `FileReference` using the `asFileReference` message. In the other direction, you can get a file URL from a
`FileReference` using the `asUrl` or `asZnUrl` messages. Do keep in mind that there is no such thing as a relative file URL, only absolute file URLs exist.
