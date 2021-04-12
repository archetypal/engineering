RFC by Tech Team

# REST API HTTP Semantics

This document provides guidance on implementing the HTTP/1.1 standard ([RFC7231](https://tools.ietf.org/html/rfc7231)).

The principal tenant of this guide is that API's are designed to work with resources which are managed using the HTTP methods (verbs).

This guide does NOT address URL design or naming standards for resource representations.

Document Organization
=====================

This document is organized into sections that provide interpretation(s) of the linked IETF specification.

For example, under **Method Definitions** below, the first header **GET** provides a link to the IETF specification with the following sections explaining how the specification is interpreted.

Method Definitions
==================

[GET](http://tools.ietf.org/html/rfc7231#section-4.3.1)
-------------------------------------------------------

The [GET](http://tools.ietf.org/html/rfc7231#section-4.3.1) method requests transfer of a representation of the target resource. The GET may be a [Range Request](https://tools.ietf.org/html/rfc7233).

A GET should return a 200 (OK) or 206 (Partial Content) with a representation for the target resource. If the resource can not be found for a properly constructed URL, a 404 (Not Found) should be returned.  If the URL contains an invalid resource name or contains an invalid query parameter, a 400 (Bad Request) should be returned.

A GET MUST NOT have a payload.  If a payload is provided, either a 400 Bad Request status code should be returned or the payload must be ignored.

Range requests should use the [Range](https://tools.ietf.org/html/rfc7233#section-3.1) header.

### Deviations

Implementations may consider use of query parameters instead of range header for a range request.

[HEAD](http://tools.ietf.org/html/rfc7231#section-4.3.2)
--------------------------------------------------------

The [HEAD](http://tools.ietf.org/html/rfc7231#section-4.3.2) method is identical to GET except that the server MUST NOT send a message body in the response.

Head is generally not implemented unless needed by a specific use case.  A workaround is to make a call GET, possibly with a limited Range Request, and ignore the result.

[POST](http://tools.ietf.org/html/rfc7231#section-4.3.3)
--------------------------------------------------------

The [POST](http://tools.ietf.org/html/rfc7231#section-4.3.3) method requests that the target resource process the representation enclosed in the request according to the resource's own specific semantics.

If POST is used for resource creation then a 201 (Created) response status along with a full representation of the resource should be returned. The [Location](https://tools.ietf.org/html/rfc7231#section-7.1.2) header should be set to indicate the location of the resource. 

There may be use cases for POST doing more than creating new resources. Expanding the use of POST will be addressed if/when those use cases arise.  The IETF specification does NOT restrict a POST to creating new resources.

#### Example

```http
POST /iot/1.0/student HTTP/1.1
Host: edge.example.com
Content-Type: application/son
{
  "name": "John Doe"
}
```

[PUT](http://tools.ietf.org/html/rfc7231#section-4.3.4)
-------------------------------------------------------

The [PUT](http://tools.ietf.org/html/rfc7231#section-4.3.4) method requests that the state of the target resource be created or replaced with the state defined by the representation enclosed in the request message payload. A successful PUT of a given representation would suggest that a subsequent GET on that same target resource will result in an equivalent representation being sent in a 200 (OK) response.

Implement PUT per the HTTP/1.1 spec.

In large resource graphs, sometimes a client may wish to only send a delta, not the complete change of the resource. Should that use case arise, PATCH should be implemented.

Another use case may arise where a new resource is created with a predefined resource id. These requests should go through a PUT, not a POST.

### [DELETE](http://tools.ietf.org/html/rfc7231#section-4.3.5)

The [DELETE](http://tools.ietf.org/html/rfc7231#section-4.3.5) method requests that the origin server remove the association between the target resource and its current functionality.

Return 204 (No Content) or 202 (Accepted) on success. However, there may be use cases where a 200 (OK) with content may be needed.

[PATCH](https://tools.ietf.org/html/rfc5789#section-2)
------------------------------------------------------

The [PATCH](https://tools.ietf.org/html/rfc5789#section-2) method requests that a set of changes described in the request entity be applied to the resource identified by the Request-URI.

Implement PATCH per the HTTP/1.1 spec. Return a representation of the entire resource as if a GET followed the PATCH call.

Successful 2xx (Success Response Codes)
=======================================

[200 OK](http://tools.ietf.org/html/rfc7231#section-6.3.1)
----------------------------------------------------------

The 200 (OK) status code indicates that the request has succeeded.

### APIs

Generally will use 200 (OK) in most situations.

The 200 (OK) status code is generally restricted to the GET method. However, there may be use cases for a 200 (OK) to be used after a POST, PUT or DELETE.

[201 Created](http://tools.ietf.org/html/rfc7231#section-6.3.2)
---------------------------------------------------------------

The 201 (Created) status code indicates that the request has been fulfilled and has resulted in one or more new resources being created.

Generally restricted to the POST method.

The 201 (Created) status code is generally restricted to a POST. However, a use case may arise where a 201 (Created) is returned from a PUT.

[202 Accepted](http://tools.ietf.org/html/rfc7231#section-6.3.3)
----------------------------------------------------------------

The 202 (Accepted) status code indicates that the request has been accepted for processing, but the processing has not been completed.

Usage is uncomon common.

Check the Link headers for a link with the relationship of "monitor".

[204 No Content](http://tools.ietf.org/html/rfc7231#section-6.3.5)
------------------------------------------------------------------

The 204 (No Content) status code indicates that the server has successfully fulfilled the request and that there is no additional content to send in the response payload body.

Restricted to PUT and DELETE methods. Even if a GET returns no content, a 200 (OK) is used.

[206 Partial Content](https://tools.ietf.org/html/rfc7233#section-4.1)
----------------------------------------------------------------------

The [206 (Partial Content)](https://tools.ietf.org/html/rfc7233#section-4.1) status code indicates that the server has fulfilled a range request.

The Content-Range response header is not required.

Client Error 4xx (Failure Response Codes)
=========================================

[400 Bad Request](http://tools.ietf.org/html/rfc7231#section-6.5.1)
-------------------------------------------------------------------

The [400 (Bad Request)](http://tools.ietf.org/html/rfc7231#section-6.5.1) status code indicates that the server cannot or will not process the request due to something that is perceived to be a client error.

In addition to an invalid parameter, may be used for invalid resource name. For example, if api/**presno**/1234 is entered, and there is no presno resource (assume it is a mistype of **person**), a 400 (Bad Request) would be returned.

[401 Unauthorized](http://tools.ietf.org/html/rfc7235#section-3.1)
------------------------------------------------------------------

The [401 (Unauthorized)](http://tools.ietf.org/html/rfc7235#section-3.1) status code indicates that the request has not been applied because it lacks valid authentication credentials for the target resource. The server generating a 401 response MUST send a WWW-Authenticate header field ([Section 4.1](http://tools.ietf.org/html/rfc7235#section-4.1)) containing at least one challenge applicable to the target resource.

[RFC 6750](http://tools.ietf.org/pdf/rfc6750.pdf) defines the response for Bearer tokens.

#### Authentication Lacking

HTTP/1.1 401 Unauthorized  
WWW-Authenticate: Bearer

#### Authentication Expired

HTTP/1.1 401 Unauthorized  
WWW-Authenticate: Bearer error="invalid_token"

[403 Forbidden](http://tools.ietf.org/html/rfc7231#section-6.5.3)
-----------------------------------------------------------------

The [403 (Forbidden)](http://tools.ietf.org/html/rfc7231#section-6.5.3) status code indicates that the server understood the request but refuses to authorize it. A server that wishes to make public why the request has been forbidden can describe that reason in the response payload (if any).

If authentication credentials were provided in the request, the server considers them insufficient to grant access. The client SHOULD NOT automatically repeat the request with the same credentials. The client MAY repeat the request with new or different credentials. However, a request might be forbidden for reasons unrelated to the credentials.

Generally not used or may use a 404 (Not Found) instead.

[404 Not Found](http://tools.ietf.org/html/rfc7231#section-6.5.4)
-----------------------------------------------------------------

The [404 (Not Found)](http://tools.ietf.org/html/rfc7231#section-6.5.4) status code indicates that the origin server did not find a current representation for the target resource or is not willing to disclose that one exists.

The 404 (Not Found) is returned from a GET or a PATCH.

[405 Method Not Allowed](http://tools.ietf.org/html/rfc7231#section-6.5.5)
--------------------------------------------------------------------------

The 405 (Method Not Allowed) status code indicates that the method received in the request-line is known by the origin server but not supported by the target resource. The origin server MUST generate an Allow header field in a 405 response containing a list of the target resource's currently supported methods.

[409 Conflict](https://tools.ietf.org/html/rfc7231#section-6.5.8)
-----------------------------------------------------------------

The [409 (Conflict)](https://tools.ietf.org/html/rfc7231#section-6.5.8) indicates that the request could not be completed due to a conflict with the current state of the target resource.

The 409 (Conflict) is used in response to a POST where the intention is to create a new resource and there is a conflict with some existing resource.

Server Error 5xx
================

[500 Internal Server Error](http://tools.ietf.org/html/rfc7231#section-6.6.1)
-----------------------------------------------------------------------------

The server encountered an unexpected condition which prevented it from fulfilling the request.

[503 Service Unavailable](https://tools.ietf.org/html/rfc7231#section-6.6.4)
----------------------------------------------------------------------------

The server is currently unable to handle the request due to a temporary overloading or maintenance of the server. The implication is that this is a temporary condition which will be alleviated after some delay. If known, the length of the delay MAY be indicated in a Retry-After header. If no Retry-After is given, the client SHOULD handle the response as it would for a 500 response.

References
==========

HTTP/1.1
--------

### [RFC 7231 - Semantics and Content](http://tools.ietf.org/html/rfc7231)

### [RFC 7235 - Authentication](http://tools.ietf.org/html/rfc7235)

[https://tools.ietf.org/html/rfc6585](https://tools.ietf.org/html/rfc6585)

  

APIs
----

Add idempotent requests: https://stripe.com/docs/api/idempotent_requests
