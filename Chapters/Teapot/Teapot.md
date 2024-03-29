{
    "metadata" : {
        "title" : "Teapot",
        "attribution" : "Attila Magyar with Johan Fabry"
    },
    "headingLevelOffset":2
}
@cha:teapot

We begin the book in this first chapter by showing how basic web applications can be written using just a few lines of code. In the second chapter we will treat the construction of web applications more in depth, also touching on the fundamentals of web application building. But we start by keeping it simple, which is possible thanks to Teapot.

Teapot is a _micro_ web framework on top of the Zinc HTTP web server described in Chapter  *@cha:zinc-server@*. It focuses on simplicity and ease of use and is itself small: around 600 lines of code, not counting unit tests. Teapot is developed by Attila Magyar and this chapter is heavily inspired from the original documentation.


# Getting Started

To get started,  execute the following code snippet, it will load the latest stable version of Teapot.

```
Gofer it
   smalltalkhubUser: 'zeroflag' project: 'Teapot'; configuration;
   loadStable.
```


It is straightforward to launch Teapot and add a page:

```
Teapot on
   GET: '/welcome' -> 'Hello World!'; start.
```


Opening a browser on [http://localhost:1701/welcome](http://localhost:1701/welcome) results in the following:

![The Teapot welcome at [http://localhost:1701/welcome](http://localhost:1701/welcome)](figures/TeapotWelcome.png width=70&label=TeapotWelcome)


## Differences between Teapot and other Web Frameworks


Teapot is not a singleton and doesn't hold any global state. You can run multiple Teapot servers inside the same image with their state being isolated from each other.

- There are no thread locals or dynamically scoped variables in Teapot. Everything is explicit.
- It doesn't rely on annotations or pragmas, the routes are defined programmatically.
- It doesn't instantiate objects \(e.g. "web controllers"\) for you. You can hook http events to existing objects, and manage their dependencies as required.




# A REST Example, Showing some CRUD Operations


Before getting into the details of Teapot. Here is a simple example for managing books. With the following code, we can list books, add a book and delete a book.

```
| books teapot |
books := Dictionary new.
teapot := Teapot configure: {
   #defaultOutput -> #json. #port -> 8080. #debugMode -> true }.
teapot
   GET: '/books' -> books;
   PUT: '/books/<id>' -> [ :req | | book |
      book := {'author' -> (req at: #author).
      'title' -> (req at: #title)} asDictionary.
      books at: (req at: #id) put: book ];
   DELETE: '/books/<id>' -> [ :req | books removeKey: (req at: #id) ];
   exception:
      KeyNotFound -> (TeaResponse notFound body: 'No such book');
   start.
```


Now you can create a book with ZnClient or your web client as follows:

```
ZnClient new
   url: 'http://localhost:8080/books/1';
   formAt: 'author' put: 'SquareBracketAssociates';
   formAt: 'title' put: 'Pharo For The Enterprise'; put
```


You can also list the contents using [http://localhost:8080/books](http://localhost:8080/books)
For a more complete example, study the `Teapot-Library-Example` package.

Now that you get the general feel of Teapot, let us see the key concepts.

# Route


The most important concept of Teapot is the Route. The template for route definitions is as follows:

```
Method : '/url/*/pattern/<param>' -> Action
```


A route has three parts:
- an HTTP method \(`GET`, `POST`, `PUT`, `DELETE`, `HEAD`, `TRACE`, `CONNECT`, `OPTIONS`, `PATCH`\),
- an URL pattern \(i.e. `/hi`, `/users/<name>`, `/foo/*/bar/*`, or a regular expression\),
- an action \(a block, message send or any object\).


In the expression below, the three rules are equivalent: 
The first one returns directly the value of an instance value; the second the value returned by the message; the third will send the message `books:` with as parameter the request as show below; the fourth will take a request as argument and execute the block. 

```
	GET: '/books' -> books;
	GET: '/books2' -> self books;
	GET: '/books3' -> (Send message: #books: to: self);
	GET: '/books4'-> [:req | self books ]
```


```
books: aRequest
    ^ books
```



Here is another example:

```
Teapot on
   GET: '/hi' -> 'Bonjour!';
   GET: '/hi/<user>' -> [:req | 'Hello ', (req at: #user)];
   GET: '/say/hi/*' -> (Send message: #greet: to: greeter); start.
```


A wildcard character \(`*`\), as in the last route, matches to one URL path segment. A wildcard terminated pattern is a greedy match; `'/foo/*'` for example matches to `'/foo/bar'` and `'/foo/bar/baz'` too.

The second route shows that the action block optionally takes the HTTP request. The third route is an example of a message send, by using the `Send` class. The selector of the message can take maximum 2 arguments, which will be instances of
 a `TeaRequest` and `TeaResponse`.

It is also possible to use the Zinc client \(see  Chapter *@cha:zinc-client@*\) to query the server. The example below illustrates the use of parameters, which we discuss next.

```
(ZnEasy get: 'http://localhost:1701/hi/user1') entity string.
   --> "Hello user1"
```


## Parameters in URLs

% %HERE
The URL pattern may contain named parameters \(e.g., `<user>` above\), whose values are accessible via the request object. The request is an extension of `ZnRequest` with some extra methods.

Query parameters and Form parameters can be accessed the same way as path parameters `(req at: #paramName)`. Teapot can perform conversions of parameters to a number, for example as follows:

```
Teapot on
   GET: '/user/<id:IsInteger>' -> [ :req |
      users findById: (req at: #id) ];
   output: #ston; start.
```


- `IsInteger` matches digits \(negative or positive\) only and converts the value to an Integer.
- `IsNumber` matches any integer or floating point number and converts the value to a Number.


See also the, `IsInteger` and `IsNumber` classes for information about introducing user defined conversions.

## Using Regular Expressions


Instead of `<` and `>` surrounded named parameters, the regexp pattern may contain subexpressions between parentheses whose values are accessible via the
request object.

The following example matches any `/hi/user` followed by two digits.

```
Teapot on
   GET: '/hi/([a-z]+\d\d)' asRegex -> [ :req | 'Hello ', (req at: 1)];
   start.

(ZnEasy get: 'http://localhost:1701/hi/user01') entity string.
   --> "Hello user01"
ZnEasy get: 'http://localhost:1701/hi/user'
   --> not found
```


## How are Routes Matched?


The routes are matched in the order in which they are defined.

The first route that matches the request method and the URL is invoked.
- If a route matches but it returns a 404 error, the search will continue.
- If no route matches, the error 404 is returned.
- If a route was invoked, its return value will be transformed to a HTTP response, e.g. if a string is returned it will be transformed to a response with the `text/html` content-type.
- If a route returns a `ZnResponse`, no transformation will be performed.
- If a route has a response transformer defined \(see below\), the specified transformation will be performed.


## Aborting


An `abort:` message sent to the request object immediately stops a request \(by signaling an exception\) within a route. For example:

```
Teapot on
   GET: '/secure/*' -> [ :req | req abort: TeaResponse unauthorized];
   GET: '/unauthorized' -> [ :req | req abort: 'go away' ]; start.
```


# Transforming Output from Actions


The default output for Teapot is HTML: the output of an action is interpreted as a string and the content-type of the HTML response is set to `text/html`. The output of an action may actually undergo any kind of transformations by a response transformer. Response Transformers have the ultimate responsibility for constructing the outgoing HTTP response \(an instance of the class `ZnResponse`\). To clarify, HTTP requests take the following path through Teapot:

```
ZnRequest -> [Router] -> TeaRequest -> [Route] -> response -> [Resp.Transformer] -> ZnResponse
```


The response returned by the action can be:
- Any Object that will be transformed by the given response transformer \(e.g., HTML, STON, JSON, Mustache, stream\) to an HTTP response \(instance of `ZnResponse`\).
- A `TeaResponse` that allows additional parameters to be added \(response code, headers\).
- A `ZnResponse` that will be handled directly by the `ZnServer` without further transformation.


For example, the following three routes produce the same output.

```
GET: '/greet' -> [:req | 'Hello World!' ]
GET: '/greet' -> [:req | TeaResponse ok body: 'Hello World!' ]
GET: '/greet' -> [:req |
   ZnResponse new
      statusLine: ZnStatusLine ok;
      entity: (ZnEntity html: 'Hello World!'); yourself ]
```


## Response Transformers


The responsibility of a response transformer is to convert the output of the action block to HTML and to set the content-type of the response.
Some response transformers require external packages \(e.g., NeoJSON, STON, Mustache\). See the `TeaOutput` class for more information, for example the HTML transformer is  `TeaOutput html`.

For example, with the following configuration:

```
Teapot on
   GET: '/jsonlist' -> #(1 2 3 4); output: #json;
   GET: '/sometext' -> 'this is text plain'; output: #text;
   GET: '/download' -> ['/tmp/afile' asFileReference readStream];
   output: #stream; start.
```


Figure *@plainText@* shows the result for the `/sometext` route.

![Teapot producing plain text [http://localhost:1701/sometext](http://localhost:1701/sometext)](figures/plainText.png width=70&label=plainText)

If the NeoJSON package is loaded \(See
chapter  *@cha:JSON@*.\) the `jsonlist` transformer will return a JSON array:

```
(ZnEasy get: 'http://localhost:1701/jsonlist') entity string.
   --> '[1,2,3,4]'"
```


If you have a file located `/tmp/afile` you can access it

```
ZnEasy get: 'http://localhost:1701/download'
   --> a ZnResponse(200 OK application/octet-stream 35B)
```


If Mustache is installed  \(See chapter  *@cha:mustache@*.\)  you can output templated information.

```
Teapot on
   GET: '/greet' -> {'phrase' -> 'Hello'. 'name' -> 'World'};
   output: (TeaOutput mustacheHtml: '<b>{{phrase}}</b> <i>{{name}}</i>!'); start.
```


# Before and After Filters


Teapot also offers before and after filters.
Before filters are evaluated before each request that matches the given URL pattern. Requests can also be aborted \(by sending the `abort:` message\) in before and after filters.

In the following example a before filter is used to enable authentication: if the session has no `#user` attribute, the browser is redirected to a login page.
```
Teapot on
   before: '/secure/*' -> [ :req |
      req session
         attributeAt: #user
         ifAbsent: [ req abort: (TeaResponse redirect location: '/loginpage')]];
   before: '*' -> (Send message: #logRequest: to: auditor);
   GET: '/secure' -> 'I am a protected string';
   start.
```


After filters are evaluated after each request and can read the request and modify the response.
```
Teapot on
   after: '/*' -> [ :req :resp |
      resp headers at: 'X-Foo' put: 'set by after filter'];
   start.
```


# Error Handlers

Teapot also handles exceptions of a configured type\(s\) for all routes and before filters.
The following example illustrates how the errors raised in actions can be captured by exception handlers.

```
Teapot on
   GET: '/divide/<a>/<b>' -> [ :req | (req at: #a) / (req at: #b)];
   GET: '/at/<key>' -> [ :req | dict at: (req at: #key)];
   exception: ZeroDivide -> [ :ex :req | TeaResponse badRequest ];
   exception: KeyNotFound -> {#result -> 'error'. #code -> 42};
   output: #json; start.
```


The request `/div/6/3` succeeds and returns 2. The request `/div/6/0` raises an error and it is caught and returns
a bad request.

```
(ZnEasy get: 'http://localhost:1701/div/6/3') entity string.
   --> 2
(ZnEasy get: 'http://localhost:1701/div/6/0').
   --> "bad request"
```


You can use a comma-separated exception set to handle multiple exceptions.

```
exception: ZeroDivide, DomainError -> handler
```


The same rules apply for the return values of the exception handler as were used for the Routes.

# HTML form processing


The POST method is commonly used for HTML form submission. When a POST request comes in the form data will be accessible via the request parameter. The request object has a generic `at:` method that can be used to access the path, query or form parameters in a uniform way.

The following example shows how to create a simple login form.

```
Teapot on
   GET: '/login' ->
      '<html>
         <form method="POST">
            User name:<br><input type="text" name="user"><br>
            Password:<br><input type="password" name="pwd"><br>
            <input type="submit" value="Submit">
         </form>
      </html>';
   POST: '/login'-> [ :req | (req at: #pwd) = 'secret' ifTrue: [ 'Welcome ', (req at: #user) ] ifFalse: [ 'Go away!' ] ];
   start.
```


The /login URL is linked to two actions, one for GET requests and another for POST requests. The first one displays a HTML login form to the user. The second action is invoked on a form submission and checks the given login credentials.

The request body can be accessed directly using `req entity contents`. This is useful when interacting with JSON or XML request body contents instead of HTML forms.

# Serving Static Files


Teapot can straightforwardly serve static files. The following example serves the files located on the file system at `/var/www/htdocs` at the `/static` URL.

```
Teapot on
   serveStatic: '/static' from: '/var/www/htdocs'; start.
```


# Conclusion

Teapot is a powerful and simple web framework. It is based on the notion of routes and request transformations. It supports the definition of REST application.

Now an important point: Where does the name come from? _418 I'm a teapot \(RFC 2324\)_ is an HTTP status code.
It was defined in 1998 as one of the traditional IETF April Fools' jokes, in RFC 2324, Hyper Text Coffee Pot Control Protocol, and is not expected to be
implemented by actual HTTP servers.

%  Local Variables:
%  tab-width: 3
%  evil-shift-width: 3
%  End: