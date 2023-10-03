## Zinc HTTP: The Client Side
   url: 'http://zn.stfx.eu/zn/small.html';
   get;
   response.
response := ZnClient new
   url: 'http://zn.stfx.eu/zn/small.html';
   get;
   response.
response writeOn: Transcript.
Transcript flush.
Date: Thu, 26 Mar 2015 23:26:49 GMT
Modification-Date: Thu, 10 Feb 2011 08:32:30 GMT
Content-Length: 113
Server: Zinc HTTP Components 1.0
Vary: Accept-Encoding
Content-Type: text/html;charset=utf-8

<html>
<head><title>Small</title></head>
<body><h1>Small</h1><p>This is a small HTML document</p></body>
</html>
request := (ZnClient new)
   url: 'http://zn.stfx.eu/zn/small.html';
   get;
   request.
request writeOn: Transcript.
Transcript flush.
Accept: */*
User-Agent: Zinc HTTP Components 1.0
Host: zn.stfx.eu
   logToTranscript;
   get: 'http://zn.stfx.eu/zn/small.html'.
2015-03-26 20:32:30 002 Request Written a ZnRequest(GET /zn/small.html) 0ms
2015-03-26 20:32:30 003 Response Read a ZnResponse(200 OK text/html;charset=utf-8 113B) 223ms
2015-03-26 20:32:30 004 GET /zn/small.html 200 113B 223ms
   'http://esug.org/data/Logos+Graphics/ESUG-Logo/2006/gif/',
   'esug-Logo-Version3.3.-13092006.gif'.
ZnEasy getJpeg: 'http://caretaker.wolf359.be/sun-fire-x2100.jpg'.
ZnEasy getPng: 'http://pharo.org/files/pharo.png'.

(ZnEasy getPng: 'http://chart.googleapis.com/chart?cht=tx&chl=',
   'a^2+b^2=c^2') asMorph openInHand.
client := ZnClient new.
client get: 'http://zn.stfx.eu/zn/numbers.txt'.
client isSuccess
   ifTrue: [ client contents lines collect: [ :each | each asNumber ] ]
   ifFalse: [ self inform: 'Something went wrong' ]
   enforceHttpSuccess: true;
   ifFail: [ :ex | self inform: 'Cannot get numbers: ', ex printString ];
   get: 'http://zn.stfx.eu/zn/numbers.txt'.
   timeout: 1;
   get: 'http://zn.stfx.eu/zn/small.html'.
   value: 5
   during: [ ^ ZnClient new get: 'http://zn.stfx.eu/zn/small.html' ].
ZnNetworkingUtils defaultSocketStreamTimeout: 60.
   numberOfRetries: 3;
   retryDelay: 2;
   get: 'http://zn.stfx.eu/zn/small.html'.
   http;
   host: 'zn.stfx.eu';
   addPath: 'zn';
   addPath: 'small.html';
   get.
   http;
   host: 'www.google.com';
   addPath: 'search';
   queryAt: 'q' put: 'Pharo Smalltalk';
   get.
   scheme: #http;
   host: 'www.google.com';
   port: 80;
   addPathSegment: 'search';
   queryAt: 'q' put: 'Pharo Smalltalk';
   yourself.
'http://www.google.com:80/search?q=Pharo Smalltalk' asZnUrl.
   Search for: <input type="text" name="search-field"/>
   <input type="submit" value="Go!"/>
</form>
   url: 'http://www.search-engine.com/search-handler';
   formAt: 'search-field' put: 'Pharo Smalltalk';
   post.
   Search for: <input type="text" name="search-field"/>
   <input type="submit" value="Go!"/>
</form>
   url: 'http://www.search-engine.com/search-handler';
   addPart: (ZnMimePart
                fieldName: 'search-field'
                value: 'Pharo Smalltalk');
   post.
   Photo file: <input type="file" name="photo-file"/>
   <input type="submit" value="Upload!"/>
</form>
   url: 'http://www.search-engine.com/upload-handler';
   addPart: (ZnMimePart
                fieldName: 'photo-file'
                fileNamed: '/Pictures/cat.jpg');
   post.
   username: 'john@hacker.com' password: 'trustno1';
   get: 'http://www.example.com/secret.txt'.
   url: 'http://cloud-storage.com/login';
   formAt: 'username' put: 'john.doe@acme.com';
   formAt: 'password' put: 'trustno1';
   post;
   get: 'http://cloud-storage.com/my-file'.
   put: 'http://zn.stfx.eu/echo' contents:'Hello there!'.

ZnClient new
   post: 'http://zn.stfx.eu/echo' contents: #[0 1 2 3 4 5 6 7 8 9].

ZnClient new
   url: 'http://zn.stfx.eu/echo';
   entity: (ZnEntity
               with: '<xml><object><id>42</id></object></xml>'
               type: ZnMimeType applicationXml);
   post.
   url: 'http://www.apache.org';
   method: #OPTIONS;
   execute;
   response.
   client := ZnClient new url: 'http://zn.stfx.eu'.
   (1 to: 10) collect: [ :each | | url |
      url := '/random/', each asString.
      stream nextPut: (client path: url; get) ].
   client close ].
   beOneShot;
   get: 'http://zn.stfx.eu/numbers.txt'.
   url: 'http://zn.stfx.eu/zn/numbers.txt';
   setIfModifiedSince: (Date year: 2011 month: 1 day: 1);
   downloadTo: FileLocator imageDirectory.

ZnClient new
   url: 'http://zn.stfx.eu/zn/numbers.txt';
   setIfModifiedSince: (Date year: 2012 month: 1 day: 1);
   get;
   response.
   enforceAcceptContentType: true;
   accept: ZnMimeType textPlain;
   get: 'http://zn.stfx.eu/zn/numbers.txt'.
   head: 'http://zn.stfx.eu/zn/small.html';
   response.
ZnClient new request setAccept: 'text/*'.
ZnClient new request headers at: 'Accept' put: 'text/*'.
ZnClient new request headers at: 'ACCEPT' put: 'text/*'.
ZnClient new request headers at: 'accept' put: 'text/*'.
(client response headers at: 'Connection' ifAbsent: [ '' ])
   sameAs: 'close'.
   systemPolicy;
   url: 'http://zn.stfx.eu/zn/numbers.txt';
   accept: ZnMimeType textPlain;
   contentReader: [ :entity |
      entity contents lines
         collect: [ :each | each asInteger ] ];
  get.
   url: 'http://internet-calculator.com/sum';
   contentWriter: [ :numberCollection |
      ZnEntity text:
         (Character space join:
            (numberCollection collect: [ :each | each asString ])) ];
   contentReader: [ :entity | entity contents asNumber ];
   post.
   url: 'http://zn.stfx.eu/zn/numbers.txt';
   downloadTo: FileLocator imageDirectory.
   url: 'http://cloudstorage.com/myfiles/';
   username: 'john@foo.co.uk' password: 'asecret';
   uploadEntityFrom: FileLocator imageDirectory / 'numbers.txt';
   post.
   bar label: 'Downloading latest Pharo image...'.
   [ ^ ZnClient new
         signalProgress: true;
         url: 'http://files.pharo.org/image/stable/latest.zip';
         downloadTo: FileLocator imageDirectory ]
   on: HTTPProgress
   do: [ :progress |
         bar label: progress printString.
         progress isEmpty ifFalse: [ bar current: progress percentage ].
         progress resume ] ]
   optionAt: #timeout
   ifAbsent: [ ZnNetworkingUtils defaultSocketStreamTimeout ]
   self
      enforceHttpSuccess: true;
      enforceAcceptContentType: true;
      numberOfRetries: 2