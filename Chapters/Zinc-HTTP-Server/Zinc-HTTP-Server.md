## Zinc HTTP: The Server Side

@cha:zinc-server


Zinc is both a client and server HTTP library written and maintained by Sven van Caekenberghe. HTTP clients and servers are each others' mirror: An HTTP client sends
a request and receives a response. An HTTP server receives a request
and sends a response. Hence the fundamental Zn framework objects are used to implement both clients and servers.

This chapter focuses on the server-side features of Zinc and
demonstrates through small, elegant and robust examples some
possibilities of this powerful library. The client side is described
in Chapter *@cha:zinc-client@*

### Running a Simple HTTP Server


Getting an independent HTTP server up and running inside a Pharo image is surprisingly easy.

```
ZnServer startDefaultOn: 1701.
```


Don't try this just yet. To be able to see what is going on, it is
better to enable logging, as follows:

```
(ZnServer defaultOn: 1701)
   logToTranscript;
   start.
```


This starts the default HTTP server, listening on port 1701. We use
1701 in the example because using a port below 1024 requires special OS level privileges, and ports like 8080 might already be in use.
Visiting [http://localhost:1701](http://localhost:1701) with a browser yields the Zn welcome
page. The Transcript produces output related to the server's
activities, for example:

```language=no language
2015-06-11 18:06:31 001 565881 Server Socket Bound 0.0.0.0:1701
2015-06-11 18:06:31 002 275888 Started ZnManagingMultiThreadedServer HTTP port 1701
2015-06-11 18:06:35 003 565881 Connection Accepted 127.0.0.1
2015-06-11 18:06:35 004 097901 Request Read a ZnRequest(GET /) 0ms
2015-06-11 18:06:35 005 097901 Request Handled a ZnRequest(GET /) 0ms
2015-06-11 18:06:35 006 097901 Response Written a ZnResponse(200 OK text/html;charset=utf-8 977B) 2ms
2015-06-11 18:06:35 007 097901 GET / 200 977B 2ms
2015-06-11 18:06:35 008 097901 Request Read a ZnRequest(GET /favicon.ico) 129ms
2015-06-11 18:06:35 009 097901 Request Handled a ZnRequest(GET /favicon.ico) 0ms
2015-06-11 18:06:35 010 097901 Response Written a ZnResponse(200 OK image/vnd.microsoft.icon 318B) 2ms
2015-06-11 18:06:35 011 097901 GET /favicon.ico 200 318B 2ms
2015-06-11 18:06:35 012 097901 Request Read a ZnRequest(GET /favicon.ico) 32ms
2015-06-11 18:06:35 013 097901 Request Handled a ZnRequest(GET /favicon.ico) 0ms
2015-06-11 18:06:35 014 097901 Response Written a ZnResponse(200 OK image/vnd.microsoft.icon 318B) 0ms
2015-06-11 18:06:35 015 097901 GET /favicon.ico 200 318B 0ms
2015-06-11 18:07:05 016 097901 Server Read Error ConnectionTimedOut: Data receive timed out.
2015-06-11 18:07:05 017 097901 Server Connection Closed 127.0.0.1
```


You can see the server starting and initializing its server socket on which it listens for incoming connections. When a connection comes in, it starts executing
its request-response loop. Then it gets a `GET` request for `/` \(the home page\), to which it answers a 200 OK response with 997 bytes of HTML. The browser also
asks for a `favicon.ico`, which the server supplies. The request-response loop is kept alive for some time and usually closes when the other end does. Although it
looks like an error, it actually is normal, expected behavior.

The example uses the default server: Zn manages a default server to
ease interactive experimentation. The server object is obtained by:
`ZnServer default`.  The default server also survives image save
and restart cycles and needs to be stopped with `ZnServer stopDefault.` The Transcript output will confirm what happens:

```language=no language
2015-06-11 18:11:07 018 565881 Server Socket Released 0.0.0.0:1701
2015-06-11 18:11:07 019 275888 Stopped ZnManagingMultiThreadedServer HTTP port 1701
```


!!note Due to its implementation, the server will print a debug notification: `Wait for accept timed out`, every 5 minutes. Again, although it looks like an error, it is by design and normal, expected behavior.

### Server Delegate, Testing and Debugging


The functional behavior of a `ZnServer` is defined by an object called its delegate. A delegate implements the key method `handleRequest:` which gets the
incoming request as parameter and has to produce a response as
result. The delegate only needs to reason in terms of a `ZnRequest`
and a `ZnResponse`. The technical side of being an HTTP server, like
the protocol itself, the networking and the \(optional\) multiprocessing, is handled by the
server object.

This allows us to write what is arguably the simplest possible HTTP
server behavior:

```
(ZnServer startDefaultOn: 1701)
   onRequestRespond: [ :request |
      ZnResponse ok: (ZnEntity text: 'Hello World!') ].
```


Now go to [http://localhost:1701](http://localhost:1701) or do:

```
ZnEasy get: 'http://localhost:1701'.
```


This server does not look at the incoming request. It always answers `200 OK` with a `text/plain` string `Hello World!`. The
`onRequestRespond:` method accepts a block that takes a request and that should produce a response. It is implemented using the helper object `ZnValueDelegate`, which
converts `handleRequest:` to `value:` on a wrapped block.

### The Default Server Delegate


Out of the box, a `ZnServer` will have a certain functionality that is related to testing and debugging. The
`ZnDefaultServerDelegate` object implements this behavior. Assuming a server is running locally on port 1701, this is the list of URLs that are available.

- [http://localhost:1701/](http://localhost:1701/) the default for `/`, equivalent to   `/welcome`
- [http://localhost:1701/bytes](http://localhost:1701/bytes) a collection of bytes
- [http://localhost:1701/dw-bench](http://localhost:1701/dw-bench) a dynamically generated page for benchmarking
- [http://localhost:1701/echo](http://localhost:1701/echo) a textual response echoing the request
- [http://localhost:1701/favicon.ico](http://localhost:1701/favicon.ico) nice Zn favicon used by browsers
- [http://localhost:1701/form-test-1](http://localhost:1701/form-test-1)  to `/form-test-3` are form test pages
- [http://localhost:1701/help](http://localhost:1701/help) this list of URLs
- [http://localhost:1701/random](http://localhost:1701/random) a random string of characters
- [http://localhost:1701/session](http://localhost:1701/session) information about the session
- [http://localhost:1701/status](http://localhost:1701/status) a textual page showing some server internals
- [http://localhost:1701/unicode](http://localhost:1701/unicode) a UTF-8 encoded page listing the first 591 Unicode characters
- [http://localhost:1701/welcome](http://localhost:1701/welcome) the standard Zn greeting page


The random handler normally returns 64 characters, you can specify your own size as well. For example, `/random/1024` will respond with a 1Kb random string.
The random pattern consists of hexadecimal digits and ends with a linefeed. The standard, slower UTF-8 encoding is used instead of the faster LATIN-1 encoding.

The bytes handler has a similar size option. Its output is in the form of a repeating BCDA pattern. When requesting equally sized byte patterns repeatably, some
extra server side caching will improve performance.

### Testing and Debugging


The echo handler is used extensively by the unit tests. It not only lists the request headers as received by the server, but even the entity if there is one. In
case of a non-binary entity, the textual contents will be included. This is really useful to debug PUT or POST requests.

In general, to help in debugging a server, enabling logging is
important to learn what is going on. Breakpoints can be put anywhere
in the server, but interrupting a running
server can sometimes be a bit hard or produce strange results. This is
because the server and its spawned handler subprocesses are different from the UI process.

When logging is enabled, the server will also keep track of the last request and response it processed. You can inspect these to find out what happened, even if
there was no debugger raised.

### Server Authenticator


Similar to the delegate, a `ZnServer` also has an authenticator object whose function is to authenticate requests. An authenticator has to implement the
`authenticateRequest:do:` method whose first argument is the incoming request and second argument a block. This method has to produce a response, like
`handleRequest:` does. If the request is allowed, the block should be evaluated, which will produce the response. If the request is denied, the authenticator should generate a 401 Unauthorized response. One simple authenticator is available to add basic HTTP authentication:

```
(ZnServer startDefaultOn: 1701)
   authenticator: (ZnBasicAuthenticator username: 'admin' password: 'secret').
```


Now, when you try to visit the server at [http://localhost:1701](http://localhost:1701) you will have to provide a username and password. Note that it is also possible to use `ZnEasy` to send
a get request to this URL with these credentials.

```
ZnEasy
   get: 'http://localhost:1701'
   username: 'admin'
   password: 'secret'.
```


!!note Using `ZnBasicAuthenticator` or implementing an alternative authenticator is only one of several possibilities to address the problem of adding security to a web site or web application.

### Logging


Log output consists of a log message preceded by a number of fixed fields. Here is an example of a server log.

```language=no language
2015-06-11 10:19:59 001 220937 Server Socket Bound 0.0.0.0:1701
2015-06-11 10:19:59 002 233075 Started ZnManagingMultiThreadedServer HTTP port 1701
2015-06-11 10:25:36 003 220937 Connection Accepted 127.0.0.1
2015-06-11 10:25:36 004 879540 Request Read a ZnRequest(GET /help) 2ms
2015-06-11 10:25:36 005 879540 Request Handled a ZnRequest(GET /help) 0ms
2015-06-11 10:25:36 006 879540 Response Written a ZnResponse(200 OK text/html;charset=utf-8 867B) 0ms
2015-06-11 10:25:36 007 879540 GET /help 200 867B 0ms
2015-06-11 10:25:38 008 879540 Request Read a ZnRequest(GET /help) 1770ms
2015-06-11 10:25:38 009 879540 Request Handled a ZnRequest(GET /help) 0ms
2015-06-11 10:25:38 010 879540 Response Written a ZnResponse(200 OK text/html;charset=utf-8 867B) 0ms
2015-06-11 10:25:38 011 879540 GET /help 200 867B 0ms
2015-06-11 10:25:44 012 879540 Request Read a ZnRequest(GET /unicode) 6082ms
2015-06-11 10:25:44 013 879540 Request Handled a ZnRequest(GET /unicode) 5ms
2015-06-11 10:25:44 014 879540 Response Written a ZnResponse(200 OK text/html;charset=utf-8 11454B) 2ms
2015-06-11 10:25:44 015 879540 GET /unicode 200 11454B 7ms
```


The first two fields are the date and time in a fixed sized format.
The next field is the id of the log entry.
The next number is a fixed sized hash of the process ID. Note how 3 different processes are
involved: the one starting the server \(probably the UI process\), the actual server listening process, and the client worker process spawned to handle the request.

Both `ZnClient` and `ZnServer` implement logging using a similar mechanism based on the announcements framework. `ZnLogEvents` are subclasses of the `Announcement` class and are
sent by an HTTP server or client containing logging information. A log event has a TimeStamp, an id, and a message.

To log something, a server or client uses its own log methods.
For example, a server receives a `logConnectionAccepted:` message with the socket that will process the request as argument. In `ZnSingleThreadedServer`, the implementation of `logConnectionAccepted:` is:

```
logConnectionAccepted: socket
   logLevel < 3 ifTrue: [ ^ nil ].
   ^ (self newLogEvent: ZnConnectionAcceptedEvent)
      address: ([ socket remoteAddress ] on: Error do: [ nil ]);
      emit
```


This logging mechnism can be easily customized by implementing subclasses of `ZnLogEvent`. For example, `ZnConnectionAcceptedEvent` is a subclass of `ZnLogEvent` customized for connection acceptation.

You can also provide your own listener for `ZnLogEvents`.
The following example shows how to log events in a file named
`zn.log`, next to the image.

```
| logger |
loggerStream := (FileSystem workingDirectory / 'zn.log') writeStream.
ZnLogEvent announcer
   when: ZnLogEvent
   do: [ :event | loggerStream lf; print: event ].
(ZnServer defaultOn: 1701) start.
```


### Server Variants and Life Cycle


The class side of `ZnServer` is actually a factory to instantiate a particular concrete `ZnServer` subclass, as can be seen in `defaultServerClass`. The
hierarchy looks as follows.

```language=no language
ZnServer
   + ZnSingleThreadedServer
      + ZnMultiThreadedServer
         + ZnManagedMultiThreadedServer
```


 `ZnServer` is an abstract class. `ZnSingleThreadedServer` implements the core server functionality. It runs in one single process, which means it can only
 handle one request at a time, making it easier to understand and debug. `ZnMultiThreadedServer` spawns a new process on each incoming request, possibly
 handling multiple request/response cycles on the same connection. `ZnManagedMultiThreadedServers` keeps explicit track of which connections are alive so that
 they can be stopped when the server stops instead of letting them die out.

Server instances can be started and stopped using `start` and `stop`. By registering a server instance, by sending it `register`, it becomes managed. That
means it will survive image save and restart. This only happens automatically with the `default` server, for other server instances it needs to be enabled manually.

The main parameter a server needs is the port on which it will listen. Additionally, you can restrict the network interface the server should listen on by
setting its `bindingAddress:` to some IP address. The default, which is `nil` or `#[0 0 0 0]`, means to listen on all interfaces. With ` #[127 0 0 1]`, the
server will not respond to requests over its normal network, but only to requests coming from the same host. This is often used to increase security while proxying.

```
(ZnServer defaultOn: 1701)
   bindingAddress: #[127 0 0 1];
   logToTranscript;
   start.
```


### Static File Server


When most people think about a web server, they imagine what is technically called static file serving. There is a directory full of HTML, image, CSS, and other files, somewhere on a machine, and the web server serves
these files over HTTP to web browser clients anywhere on the network. This is indeed what Apache does in its most basic form.

Zn can do this by using a `ZnStaticFileServerDelegate`. Given a directory and an optional prefix, this delegate will serve all files it finds in that directory, for example:

```
(ZnServer startDefaultOn: 1701)
   delegate: (
      ZnStaticFileServerDelegate new
         directory: '/var/www' asFileReference;
         prefixFromString: 'static-files';
         yourself).
```


If we suppose the contents of `/var/www` is

- index.html
- small.html


You can access these files with these URLs

- [http://localhost:1701/static-files/index.html](http://localhost:1701/static-files/index.html)
- [http://localhost:1701/static-files/small.html](http://localhost:1701/static-files/small.html)


The prefix is added in front of all files being served, the actual directory where the files reside is of course invisible to the end web user. If no prefix is specified, the files will be served directly.

Note how all other URLs result in a 404 Not found error. Note that while the `ZnStaticFileServerDelegate` is very simple, it does have a couple of capabilities. Most importantly, it will do what most
people expect with respect to directories. Consider the following URLs:

- [http://localhost:1701/static-files](http://localhost:1701/static-files)
- [http://localhost:1701/static-files/](http://localhost:1701/static-files/)


The first URL above will result in a redirect to the second. The second URL will look for either an `index.html` or `index.htm` file and serve that. Automatic
generation of an index page when there is no index file is not implemented.

As a static file server, the following features are implemented:

- automatic determination of the content mime-type based on the file extension
- correct setting of the content length based on the file length
- usage of streaming
- addition of correct modification date based on the files' last modification date
- correct reaction to the if-modified-since protocol
- optional expiration and caching control


Here is a more complex example:

```
(ZnServer startDefaultOn: 1701)
   logToTranscript;
   delegate: (
      ZnStaticFileServerDelegate new
         directory: '/var/www' asFileReference;
         mimeTypeExpirations: ZnStaticFileServerDelegate defaultMimeTypeExpirations;
         yourself);
   authenticator: (
      ZnBasicAuthenticator username: 'admin' password: 'secret').
```


In the above example, we add the optional expiration and caching control based on default settings. Note that it is easy to combine static file serving with logging and authentication.

%  JF: Commented out because not in the scope of the book
%  !!Monticello repository server

%  Another, interesting, proof of concept example is ==ZnMonticelloServerDelegate==. It allows you to run your own Smalltalk source code repository. You can try it out as follows.

%  [[[
%  (ZnServer startDefaultOn: 1701)
%     delegate: (ZnMonticelloServerDelegate new
%                  directory: '/Users/sven/Tmp/monticello' asFileReference;
%                  yourself).
%  ]]]

%  Now create an MCHttpRepository pointing to your own server.

%  [[[
%  MCHttpRepository
%    location: 'http://localhost:1701'
%    user: ''
%    password: ''.
%  ]]]

%  It should work just like any other repository, although it is for package storage only. What makes this example interesting is that it implements 3 different kinds of requests:

%  - list the repository, a GET for /
%  - get a version, a GET for a specific file
%  - store a version, a PUT of a specific file

### Dispatching


Dispatching or routing is HTTP application server speak for deciding what part of the software will handle an incoming request. This decision can be made on
any of the properties of the request: the HTTP method, the URL or part of it, the query parameters, the meta headers and the entity body. Different applications
will prefer different kinds of solutions to this problem.

Zinc HTTP Components is a general framework that offers all the necessary components to build your own dispatcher. Out of the box, there are the different
delegates that we discussed before. Most of these have hand coded dispatching in their `handleRequest:` method.

`ZnDefaultServerDelegate` can be configured to perform dispatching as it uses a prefix map internally that maps URI prefixes to internal methods. Configuration is by installing a block as the value to a prefix, which accepts the request and produces a response. Here is an example of using that capability:

```
| staticFileServerDelegate |

ZnServer startDefaultOn: 8080.

(staticFileServerDelegate := ZnStaticFileServerDelegate new)
   prefixFromString: 'zn';
   directory: '/home/ubuntu/zn' asFileReference.

ZnServer default delegate prefixMap
   at: 'zn'
   put: [ :request | staticFileServerDelegate handleRequest: request ];
   at: 'redirect-to-zn'
   put: [ :request | ZnResponse redirect: '/zn/index.html' ];
   at: '/'
   put: 'redirect-to-zn'.
```


This is taken from the configuration of what runs at [http://zn.stfx.eu](http://zn.stfx.eu). A static web server is set up under the `zn` prefix pointing to the directory
`/home/ubuntu/zn`. The prefix map of the default delegate is kept as is, with its standard functionality, but is modified, such that

- anything with a `zn` prefix is directly forwarded to the static file server
- a special `redirect-to-zn` prefix is set up which will issue a redirect to `/zn/index.html`
- the default `/` handler is linked to `redirect-to-zn` instead of the default `welcome:`


Another option is to use `ZnDispatcherDelegate`.

```
(ZnServer startDefaultOn: 9090) delegate: (
   ZnDispatcherDelegate new
      map: '/hello'
      to: [ :request :response |
            response entity: (ZnEntity html: '<h1>hello!</h1>') ]).
```


You configure the dispatcher using `map:to:` methods. First argument is the prefix, second argument is a block taking two arguments: the incoming request and
an already instantiated response.

### Character Encoding


Proper character encoding and decoding is crucial in today's international world. Pharo  encodes characters and strings using Unicode. The primary
internet encoding is UTF-8, but a couple of others are used as well. To translate between these two, a concrete `ZnCharacterEncoding` subclass like `ZnUTF8Encoder` is used.

`ZnCharacterEncoding` is an extension and reimplementation of regular `TextConverter`. It only works on binary input and generated binary output and it adds the
ability to compute the encoded length of a source character, a crucial operation for HTTP. It is more correct and will throw proper exceptions when things go wrong.

Character encoding is mostly invisible. Here are some code snippets using the encoders directly, feel free to substitute any Unicode character to make the test more interesting.

```
| encoder string |
encoder := ZnUTF8Encoder new.
string := 'any Unicode'.
self assert: (encoder decodeBytes: (encoder encodeString: string)) equals: string.
encoder encodedByteCountForString: string.
```


There are no automatic conversions in Zinc, so no defaults are assumed. Instead you should specify a proper Content-Type header including the charset information. Otherwise Zinc has no chance of knowing what to use and the default NullEncoder will make your string wrong.

Consider the following example:

```
ZnServer startDefaultOn: 1701.

ZnClient new
   url: 'http://localhost:1701/echo';
   entity: (ZnEntity with: 'An der schönen blauen Donau');
   post.

ZnClient new
   url: 'http://localhost:1701/echo';
   entity: (
      ZnEntity
         with: 'An der schönen blauen Donau'
        type: (ZnMimeType textPlain charSet: #'iso-8859-1'; yourself));
   post;
   yourself.
```


In the first case, a UTF-8 encoded string is POST-ed and correctly returned \(in a UTF-8 encoded response\).

In the second case, an ISO-8859-1 encoded string is POST-ed and correctly returned \(in a UTF-8 encoded response\).

In both cases the decoding was done correctly, using the specified charset \(if that is missing, the ZnNullEncoder is used\). Now, ö is not a perfect test example because its Unicode encoding value is 246 in decimal, U+00F6 in hex, still fits in 1 byte and hence survives null encoding/decoding \(it would not be the case with € for example\). That is why the following still works, although it is wrong to drop the charset.

```
ZnClient new
   url: 'http://localhost:1701/echo';
   entity: (
      ZnEntity
         with: 'An der schönen blauen Donau'
         type: (ZnMimeType textPlain clearCharSet; yourself));
   post;
   yourself.
```


### Resource Protection Limits, Content and Transfer Encoding


Internet facing HTTP servers will come under attack by malicious clients.
Good security is thus important.
The first step is a correct and safe implementation of the HTTP protocol. 
Another way a server protects itself is by implementing some resource limits.

Zinc HTTP Components currently implements and enforces the following limits:
- maximumLineLength \(4Kb\), impacting mainly the size of a header pair
- maximumEntitySize \(16Mb\), the size of incoming entities
- maximumNumberOfDictionaryEntries \(256\), which is used in headers, URLs and some entities


Of course these values may be customized if one needs to.

Also, Zn implements two important techniques used by HTTP servers when they send entity bodies to clients: Gzip encoding and chunked transfer encoding.
The first one adds compression.
The second one is used when the size of an entity is not known up front.
Instead chunks of certain sizes are sent until the entity is complete.

All this is handled internally and invisibly. The main object dealing with content and transfer encoding is `ZnEntityReader`. When necessary, the binary socket
stream is wrapped with either a `ZnChunkedReadStream` and/or a `GZipReadStream`. Zn also makes use of a `ZnLimitedReadStream` to make sure there is no read
beyond the boundaries of one single request's body, provided the content length is set.

### Seaside Adaptor


[Seaside](http://www.seaside.st/) is a well known, cross platform, advanced web application framework. It does not provide its own HTTP server but
relies on an existing one by means of an adaptor. It works well with Zn, through the use of a `ZnZincServerAdaptor`. It comes already included with certain Seaside distributions and on Pharo it is the default.

Starting this adaptor can be done using the Seaside Control panel in the normal way. Alternatively, the adaptor can be started programmatically.

```
ZnZincServerAdaptor startOn: 8080.
```


Since Seaside does its own character conversions, the Zn adaptor is configured to work in binary mode for maximum efficiency. There is complete support for POST
and PUT requests with entities in form URL, multipart or raw encoding.

There is even a special adaptor that combines being a Seaside adaptor with static file serving, which is useful if you don't like the WAFileLibrary machinery
and prefer plain static files served directly.

```
ZnZincStaticServerAdaptor startOn: 8080 andServeFilesFrom: '/var/www/'.
```


### Scripting a REST Web Service with Zinc


As a last example of the use of Zinc HTTP, we now show the implementation of REST web services, both the client and the server parts. REST or
[Representational State Transfer](http://en.wikipedia.org/wiki/Representational_state_transfer) is an architectural style most easily described as using HTTP
verbs and URIs to deal with encoded resources. Some kind of framework is needed to successfully implement a non-trivial REST service. There is one available in
the Zinc-REST-Server package, for example. Here we will implement a very small, simplified example by hand, for educational purposes.

The service will allow arbitrary [JSON](http://www.json.org/) objects to be stored on the server, each identified by an URI allocated by the server. Here is the REST API exposed by the server:



`GET /`
Returns a list of all known stored object URIs;

`GET /n`
Returns the JSON object known under URI `/n`;

`POST /`
Creates a new entry with JSON as contents, returns the new URI;

`PUT /n`
Updates \(replaces\) the contents of an existing JSON object known under URI `/n`;

`DELETE /n`
Removes the JSON object known under URI `/n`.

### The Server Code


A proper implementation should best use a couple of classes.
However for brevity, the following implementation is written in a workspace, not using any classes.
It requires STON and starts by creating two global variables to hold the stored objects and the last ID used.
The former is a standard dictionary mapping string URIs to objects.

```
JSONStore := Dictionary new.
ServerLastId := 0.
```


The server implementation uses two helper objects: a `jsonEntityBuilder` and a `mapper`.
Both make use of block closures.

```
| jsonEntityBuilder mapper |

jsonEntityBuilder := [ :object |
   ZnEntity
      with: ((String streamContents: [ :stream |
         STON jsonWriter
            on: stream;
            prettyPrint: true;
            nextPut: object.
         stream cr ])
         replaceAll: Character cr with: Character lf)
      type: ZnMimeType applicationJson ].
```


The `jsonEntityBuilder` block helps in transforming objects to a JSON entity.
We use the STON writer and reader here because they are backwards compatible with JSON.
We use linefeeds to improve compatibility with internet conventions as well as pretty printing to help human interpretation of the data.

```
mapper := {
   [ :request |
      request uri isSlash and: [ request method = #GET ] ]
   ->
   [ :request |
      ZnResponse
         ok: (jsonEntityBuilder value: JSONStore keys asArray) ].
"----------------------------------------------------------------"
   [ :request |
      request uri pathSegments size = 1 and: [ request method = #GET ] ]
   ->
   [ :request | | uri |
      uri := request uri pathPrintString.
      JSONStore
         at: uri
         ifPresent: [ :object |
            ZnResponse ok: (jsonEntityBuilder value: object) ]
         ifAbsent: [ ZnResponse notFound: uri ] ].
"----------------------------------------------------------------"
   [ :request |
      (request uri isSlash
         and: [ request method = #POST ])
         and: [ request contentType = ZnMimeType applicationJson ] ]
   ->
   [ :request | | uri |
      uri := '/', (ServerLastId := ServerLastId + 1) asString.
      JSONStore at: uri put: (STON fromString: request contents).
      (ZnResponse created: uri)
         entity: (jsonEntityBuilder value: 'Created ', uri);
         yourself ].
"----------------------------------------------------------------"
   [ :request |
      (request uri pathSegments size = 1
         and: [ request method = #PUT ])
         and: [ request contentType = ZnMimeType applicationJson ]]
   ->
   [ :request | | uri |
      uri := request uri pathPrintString.
      (JSONStore includesKey: uri)
         ifTrue: [
            JSONStore
               at: uri
               put: (STON fromString: request contents).
            ZnResponse ok: (jsonEntityBuilder value: 'Updated') ]
         ifFalse: [ ZnResponse notFound: uri ] ].
"----------------------------------------------------------------"
   [ :request |
      request uri pathSegments size = 1
         and: [ request method = #DELETE ] ]
   ->
   [ :request | | uri |
      uri := request uri pathPrintString.
      (JSONStore removeKey: uri ifAbsent: [ nil ])
         ifNil: [ ZnResponse notFound: uri ]
         ifNotNil: [
            ZnResponse ok: (jsonEntityBuilder value: 'Deleted') ] ].
}.
```



The mapper is a dynamically created array of associations \(not a dictionary\). Each association consists of two blocks. The first block is a condition: it tests a request and returns true when it matches. The second block is a handler that is evaluated with the incoming request to produce a response \(if and only if the first condition matched\).

The associations in the mapper follow exactly the list of the REST API as shown earlier. The server is set up with a block based delegate using the
`onRequestRepond:` method. Again, a more object-oriented implementation would use a proper delegate object here, but for this example, the block is sufficient.

The server logic thus becomes: find a matching entry in the mapper and invoke it. If no matching entry is found, we have a bad request. Error handling is of course rather limited in this small example.

```
(ZnServer startDefaultOn: 1701)
   logToTranscript;
   onRequestRespond: [ :request |
      (mapper
            detect: [ :each | each key value: request ]
            ifNone: [ nil ])
         ifNil: [ ZnResponse badRequest: request ]
         ifNotNil: [ :handler | handler value value: request ] ].
```


### Using the Server


Here is an example command line session using the Unix utility [curl](http://en.wikipedia.org/wiki/CURL), interacting with the server.

```language=bash
$ curl http://localhost:1701/
[ ]

$ curl -X POST -d '[1,2,3]' -H'Content-type:application/json' http://localhost:1701/
"Created /1"

$ curl http://localhost:1701/1
[
    1,
    2,
    3
]

$ curl -X POST -d '{"bar":-2}' -H'Content-type:application/json' http://localhost:1701/
"Created /2"

$ curl http://localhost:1701/2
{
    "bar" : -2
}

$ curl -X PUT -d '{"bar":-1}' -H'Content-type:application/json' http://localhost:1701/2
"Updated /2"

$ curl http://localhost:1701/2
{
    "bar" : -1
}

$ curl http://localhost:1701/
[
    "/1",
    "/2"
]

$ curl -X DELETE http://localhost:1701/2
"Deleted /2"

$ curl http://localhost:1701/2
Not Found /2
```


### A Zinc Client


It is trivial to use `ZnClient` to have the same interaction. But we can do better: using a contentWriter and contentReader, we can customise the client to do the JSON conversions automatically.

```
| client |

client := ZnClient new
   url: 'http://localhost:1701';
   enforceHttpSuccess: true;
   accept: ZnMimeType applicationJson;
   contentWriter: [ :object |
      ZnEntity
         with: (String streamContents: [ :stream |
                  STON jsonWriter on: stream; nextPut: object ])
         type: ZnMimeType applicationJson ];
   contentReader: [ :entity |  STON fromString: entity contents ];
   yourself.
```


Now we can hold the same conversation as above, only in this case in terms of real  objects.

```
client get: '/'
>>> #()

client post: '/' contents: #(1 2 3)
>>> 'Created /1'

client get: '/1'
>>> #(1 2 3)

client post: '/' contents: (Dictionary with: #bar -> -2)
>>> 'Created /2'

client put: '/2' contents: (Dictionary with: #bar -> -1)
>>> 'Updated'

client get: '/2'
>>> a Dictionary('bar'->-1 )

client get: '/'
>>> #('/1' '/2')

client delete: '/2'
>>> 'Deleted'

client get: '/2'
>>> throws a ZnHttpUnsuccessful exception
```


### Conclusion


Zinc HTTP Components was written with the explicit goal of allowing users to explore the implementation. The test suite contains many examples that can serve as learning material. This carefulness while writing Zinc HTTP Components code now enable users to customize it to their need or to build on top of it. Zinc is indeed an extremely malleable piece of software.
