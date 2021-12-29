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
