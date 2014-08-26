# Request and Response

We don't want to duplicate the work done on the documentation 
of Aura.Web library, but at the same time, people who are new to 
Aura Framework probably don't know about 
[Request](https://github.com/auraphp/Aura.Web/blob/develop-2/README-REQUEST.md) and
[Response](https://github.com/auraphp/Aura.Web/blob/develop-2/README-RESPONSE.md)
objects introduced in the previous chapters.

A basic overview of the available objects and sub-objects is given below. 
For a detailed information consider reading 
[Request](https://github.com/auraphp/Aura.Web/blob/develop-2/README-REQUEST.md) and
[Response](https://github.com/auraphp/Aura.Web/blob/develop-2/README-RESPONSE.md).

## REQUEST

The _Request_ object has these sub-objects:

- [$request->cookies](https://github.com/auraphp/Aura.Web/blob/develop-2/README-REQUEST.md#superglobals) for $_COOKIES
- [$request->env](https://github.com/auraphp/Aura.Web/blob/develop-2/README-REQUEST.md#superglobals) for $_ENV
- [$request->files](https://github.com/auraphp/Aura.Web/blob/develop-2/README-REQUEST.md#superglobals) for $_FILES
- [$request->post](https://github.com/auraphp/Aura.Web/blob/develop-2/README-REQUEST.md#superglobals) for $_POST
- [$request->query](https://github.com/auraphp/Aura.Web/blob/develop-2/README-REQUEST.md#superglobals) for $_GET
- [$request->server](https://github.com/auraphp/Aura.Web/blob/develop-2/README-REQUEST.md#superglobals) for $_SERVER
- [$request->client](https://github.com/auraphp/Aura.Web/blob/develop-2/README-REQUEST.md#client) for the client making the
  request
- [$request->content](https://github.com/auraphp/Aura.Web/blob/develop-2/README-REQUEST.md#content) for the raw body of the
  request
- [$request->headers](https://github.com/auraphp/Aura.Web/blob/develop-2/README-REQUEST.md#headers) for the request headers
- [$request->method](https://github.com/auraphp/Aura.Web/blob/develop-2/README-REQUEST.md#method) for the request method
- [$request->accept](https://github.com/auraphp/Aura.Web/blob/develop-2/README-REQUEST.md#accept) for content negotiation
- [$request->params](https://github.com/auraphp/Aura.Web/blob/develop-2/README-REQUEST.md#params) for path-info parameters
- [$request->url](https://github.com/auraphp/Aura.Web/blob/develop-2/README-REQUEST.md#url) for the request URL

## RESPONSE

The _Response_ object has these sub-objects:

- [$response->status](https://github.com/auraphp/Aura.Web/blob/develop-2/README-RESPONSE.md#status) for the status code, status
  phrase, and HTTP version
- [$response->headers](https://github.com/auraphp/Aura.Web/blob/develop-2/README-RESPONSE.md#headers) for non-cookie headers
- [$response->cookies](https://github.com/auraphp/Aura.Web/blob/develop-2/README-RESPONSE.md#cookies) for cookie headers
- [$response->content](https://github.com/auraphp/Aura.Web/blob/develop-2/README-RESPONSE.md#content) for describing the response
  content, and for convenience methods related to content type, charset,
  disposition, and filename
- [$response->cache](https://github.com/auraphp/Aura.Web/blob/develop-2/README-RESPONSE.md#cache) for convenience methods related
  to cache headers
- [$response->redirect](https://github.com/auraphp/Aura.Web/blob/develop-2/README-RESPONSE.md#redirect) for convenience methods
  related to Location and Status
