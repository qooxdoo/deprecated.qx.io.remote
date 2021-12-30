# qx.io.remote 

This qooxdoo package contains the `qx.io.remote` namespace, which has been
removed in v7 of the framework. Applications relying on classes in this
namespace can reinstall it using the command `qx package install
qooxdoo/deprecated.qx.io.remote`.

The `qx.io.remote` namespace provides classes for remote communication, i.e.
communication of the client with a server. Bear in mind that this usually
means communication with the server the client application was loaded from.
Cross-domain communication on the other hand has to be treated specially.

In the most common cases the `qx.io.remote.Request` class is the
class you will be working with. It allows you to set up a request for a
remote resource, configure request data and request processing details, and
set up event handlers for typical stages of the request processing. A brief
example:

```javascript
var req = new qx.io.remote.Request("/my/resource/url.txt");
req.addListener("completed", function (e) {
  alert(e.getContent());
});
req.send();
```

Event handlers are essential for obtaining the outcome of a request. The
parameter passed into the event handler (`e` in our example) is of type
`qx.io.remote.Response`, which provides you with various methods to inspect the
outcome of your request and retrieve response data. Internally, requests are
managed using a `qx.io.remote.RequestQueue` class. The RequestQueue is a
singleton and there is no need to deal with it in client code directly.

The `qx.io.remote.Rpc` class provides you with another high-level
interface to server interaction.  You will usually use this class if you have
a server-based "service" that you want to make accessible on the client side
via an RPC-like interface. So this class will be especially interesting for
providing a general interface that can be used in various places of the
application code through a standard API.

## Architecture

On a technical level of data exchange with the server, the
`qx.io.remote.tranport.*` classes implement different schemes.  Common features
of these transport classes are collected in the
`qx.io.remote.transport.Abstract` class, and `qx.io.remote.transport.Iframe
IframeTransport`, `qx.io.remote.transport.Script` and
`qx.io.remote.transport.XmlHttp` specialize them, depending of their interaction
model with the server. Usually, you will use one of these classes to tailor the
implementation details of a specific client-server communication in your
application. Mind that the IframeTransport and ScriptTransport classes should
not be used directly by client programmers. It is recommended to provide a
subclass implementation to make use of them.

The connection between your Request object and a specific Transport is
established through a `qx.io.remote.Exchange` object. An Exchange object can be
bound to the `qx.io.remote.Request#transport` `.transport` property of a
Request, and takes care that the particular request is realized over the
specific Transport. This allows you to accommodate a wide variety of transport
options without overloading the Request object with the details.

This system is (as everything else in Qooxdoo) completely event based. It
currently supports communication by **XMLHttp**, **Iframes** or **Script**. The
system wraps most of the differences between the implementations and unifies
them for the user/developer.

For all your communication needs you need to create a new instance of Request:

```javascript
const req = new qx.io.remote.Request(url, "GET", "text/plain");
```

Constructor arguments of Request:

1.  URL: Any valid http/https/file URL
2.  Method: You can choose between POST and GET.
3.  Response mimetype: What mimetype do you await as response

## Mimetypes supported

- application/xml
- text/plain
- text/html
- text/javascript
- application/json

> :memo: `text/javascript` and `application/json` will be directly evaluated. As
> content you will get the return value.

If you use the iframe transport implementation the functionality of the type is
more dependent on the server side response than for the XMLHttp case. For
example the text/html mimetypes really need the response in HTML and can't
convert it. This also depends greatly on the mimetype sent out by the server.

## Request data

- `setRequestHeader(key, value)`: Setup a request header to send.
- `getRequestHeader(key)`: Returns the configured value of the request header.
- `setParameter(key, value)`: Add a parameter to send with your request.
- `getParameter(key)`: Returns the value of the given parameter.
- `setData(value)`: Sets the data which should be sent with the request (only
  useful for POST)
- `getData()`: Returns the data currently set for the request

> :memo: Parameters are always sent as part of the URL, even if you select POST. If you
> select POST, use the setData method to set the data for the request body.

## Request configuration (properties)

- `asynchronous`: Should the request be asynchronous? This is `true` by default.
  Otherwise it will stop the script execution until the response was received.

- `data`: Data to send with the request. Only used for POST requests. This is
  the actual post data. Generally this is a string of url-encoded key-value
  pairs.

- `username`: The user name to authorize for the server. Configure this to
  enable authentication.

- `password`: The password to authenticate for the server.

- `timeout`: Configure the timeout in milliseconds of each request. After this
  timeout the request will be automatically canceled.

- `prohibitCaching`: Add a random numeric key-value pair to the url to securely
  prohibit caching in IE. Enabled by default.

- `crossDomain`: Enable/disable cross-domain transfers. This is `false` by
  default. If you need to acquire data from a server of a different domain you
  would need to setup this as `true`. (**Caution:** this would switch to
  "script" transport, which is a security risk as you evaluate code from an
  external source. Please understand the security issues involved.)

- `fileUpload`: Indicate that the request will be used for a file upload. The
  request will be used for a file upload. This switches the concrete
  implementation that is used for sending the request from
  `qx.io.remote.transport.XmlHttp` to `qx.io.remote.IFrameTransport`, because
  only the latter can handle file uploads.

## Available events

- `sending`: Request was configured and is sending data to the server.
- `receiving`: The client receives the response of the server.
- `completed`: The request was executed successfully.
- `failed`: The request failed through some reason.
- `timeout`: The request has got a timeout event.
- `aborted`: The request was aborted.

The last four events give you a `qx.event.type.Data` as the first parameter of
the event handler. As always for `qx.event.type.Data` you can access the stored
data using `getData()`. The return value of this function is an instance of
`qx.io.remote.Response`.

## Response object

The response object `qx.io.remote.Response` stores all the returning data of a
`qx.io.remote.Request`. This object comes with the following methods:

- `getContent`: Returns the content data of the response. This should be the
  type of content you acquired using the request.

- `getResponseHeader`: Returns the content of the given header entry.

- `getResponseHeaders`: Return all available response headers. This is a
  hash-map using typical key-values pairs.

- `getStatusCode`: Returns the HTTP status code.

> :memo: Response headers and status code information are not supported for iframe
> based communication!

## Simple example

```javascript
// get text from the server
const req = new qx.io.remote.Request(val.getLabel(), "GET", "text/plain");
// request a javascript file from the server
// req = new qx.io.remote.Request(val.getLabel(), "GET", "text/javascript");

// Switching to POST
// req.setMethod("POST");
// req.setRequestHeader("Content-Type", "application/x-www-form-urlencoded");

// Adding parameters - will be added to the URL
// req.setParameter("test1", "value1");
// req.setParameter("test2", "value2");

// Adding data to the request body
// req.setData("foobar");

// Force to testing iframe implementation
// req.setCrossDomain(true);

req.addListener("completed", function (e) {
  alert(e.getContent());
  // use the following for Qooxdoo versions <= 0.6.7:
  // alert(e.getData().getContent());
});

// Sending
req.send();
```

## Cross-Domain Requests

Sending cross-domain requests, i.e. sending a request to a URL with a domain
part other than the domain of the current document, require special treatment
since the security concept of most browsers restrict such requests.

Currently, those requests are realized through the dynamic insertion of a
"script" tag into the current document (this is the aforementioned 
`qx.io.remote.transport.Script`). The `src` attribute of the
script tag is set to the requested URL. On insertion of the script tag the
browser will load the source URL and parse and execute the returned content
as JavaScript.  This means that the returned data has to be valid JavaScript!

In order to do that and to link the completion of the script transport to your
normal request "completed" event handler, it is best that the server wraps the
return data in a call to the `qx.io.remote.transport.Script#_requestFinished`
static.  Additional to the response data, this method takes a script transport
id as a parameter, available to the server side as the "_ScriptTransport_id"
request variable. (Normal GET or POST data of the request is available through
the `_ScriptTransport_data` variable). In the response data, you also have to
take care of proper string escaping.

So the request you might see in your server log from a script transport
may look like this:
```
"GET /cgi-bin/qxresponse.cgi?_ScriptTransport_id=10&_ScriptTransport_data=action%3Ddoit HTTP/1.1" 200 -
```
and the string you return as the response might look like this:
```javascript
'qx.io.remote.transport.Script._requestFinished(10, "Thank you for asking");'
```
