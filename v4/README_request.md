1. [System](README_system.md) (white)
1. [Request](README_request.md) (blue)
1. [Accept](README_accept.md) (green)
1. [Retrieve](README_retrieve.md) (white)
1. [Precondition](README_precondition.md) (yellow)
1. Create/Process
    * [Create](README_create.md) (violet)
    * [Process](README_process.md) (red)
1. [Response](README_response.md) (cyan)
1. [Error](README_error.md) (gray)

![HTTP headers status](http-headers-status-v4.png)

## Request

This block is in charge of request-level checks.

 | callback | output | default
:-- | ---: | :--- | :---
B11 | [`method :var`](#method-var) | *Method* | `Operation.method`
 | [`allowed_methods :var`](#allowed_methods-var) | [ *Method* ] | [ OPTIONS<br>, HEAD<br>, GET<br>, POST<br>, PATCH<br>, PUT<br>, DELETE<br>, TRACE<br>]
 | [`is_method_allowed : in`](#is_method_allowed--in) | T / F |
B10 | [`is_authorized :bin`](#is_authorized-bin) | T / F | TRUE
 | [`auth_challenges :var`](#auth_challenges-var) | [ *AuthChallenge* ] | [ ]
B9 | [`method :var`](#method-var) | *Method* | `Operation.method`
 | [`is_method_trace : in`](#is_method_trace--in) | T / F |
 | [`trace_sensitive_headers :var`](#trace_sensitive_headers-var) | [ *HeaderName* ] | [ Authentication<br>, Cookies<br>]
 | [`process_trace : in`](#process_trace--in) | |
B8 | [`method :var`](#method-var) | *Method* | `Operation.method`
 | [`is_method_options : in`](#is_method_options--in) | T / F |
 | [`options_headers :var`](#options_headers-var) | { *Header*<br>: *Value* } | { Allow<br>: `allowed_methods :var`<br>, Accept-Patch<br>: `patch_content_types_accepted :var`}
 | [`process_options : in`](#process_options--in) | |
B7 | [`payload_exists : in`](#payload_exists--in) | T / F |
B6 | [`is_payload_too_large :bin`](#is_payload_too_large-bin) | T / F | TRUE
B5 | [`post_content_types_accepted :var`](#content_types_accepted-var) | { *CT*<br>: *Handler* } | { }
 | [`patch_content_types_accepted :var`](#content_types_accepted-var) | { *CT*<br>: *Handler* } | { }
 | [`put_content_types_accepted :var`](#content_types_accepted-var) | { *CT*<br>: *Handler* } | { }
 | [`content_types_accepted :var`](#content_types_accepted-var) | { *CT*<br>: *Handler* } | { }
 | [`is_content_type_accepted : in`](#is_content_type_accepted--in) | T / F |
B4 | [`content_types_accepted:handler :bin`](#content_types_accepted-handler-bin) | T / F |
B3 | [`is_forbidden :bin`](#is_forbidden-bin) | T / F | FALSE
B2 | [`is_request_ok :bin`](#is_request_ok-bin) | T / F | TRUE


### `allowed_methods :var`

Return a list of allowed methods for this resource.

### `is_method_allowed : in`

Return TRUE if `Operation.method` in `allowed_methods :var`; return FALSE otherwise.

Reference: [HTTPbis](http://tools.ietf.org/html/draft-ietf-httpbis-p2-semantics-22#section-6.5.5), [RFC2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.6)

> The 405 (Method Not Allowed) status code indicates that the method specified in the request-line is known by the origin server but not supported by the target resource.  The origin server MUST generate an Allow header field in a 405 response containing a list of the target resource's currently supported methods.

> A 405 response is cacheable unless otherwise indicated by the method definition or explicit cache controls (see Section 4.1.2 of [Part6]).

### `is_authorized :bin`

Return TRUE if this request has valid authentication credentials; return FALSE otherwise.

Reference: [HTTPbis](http://tools.ietf.org/html/draft-ietf-httpbis-p7-auth-22#section-3.1), [RFC2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.2)

> The 401 (Unauthorized) status code indicates that the request has not been applied because it lacks valid authentication credentials for the target resource.  The origin server MUST send a WWW-Authenticate header field (Section 4.4) containing at least one challenge applicable to the target resource.  If the request included authentication credentials, then the 401 response indicates that authorization has been refused for those credentials.  The client MAY repeat the request with a new or replaced Authorization header field (Section 4.1).  If the 401 response contains the same challenge as the prior response, and the user agent has already attempted authentication at least once, then the user agent SHOULD present the enclosed representation to the user, since it usually contains relevant diagnostic information.

### `auth_challenges :var`

If `is_authorized :bin` returned FALSE, then you must return a list of at least one challenge to be used as the _WWW-Authenticate_ response header.

### `is_method_trace : in`

Return TRUE if the `method :var` is TRACE; FALSE otherwise.

### `trace_sensitive_headers :var`

Return a list of headers that should be treated as sensitive, and thus hidden from the TRACE response.

### `process_trace : in`

Set `Operation.response.headers.content-type` to `message/http` and `Operation.response.body` to `Operation.headers` (except `trace_sensitive_headers :var`).

Return TRUE if succeeded; return FALSE otherwise.

Reference: [HTTPbis](http://tools.ietf.org/html/draft-ietf-httpbis-p2-semantics-22#section-4.3.8), [RFC2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.8)

> The TRACE method requests a remote, application-level loop-back of the request message.  The final recipient of the request SHOULD reflect the message received, excluding some fields described below, back to the client as the message body of a 200 (OK) response with a Content-Type of "message/http" (Section 7.3.1 of [Part1]).  The final recipient is either the origin server or the first server to receive a Max-Forwards value of zero (0) in the request (Section 5.1.2).

> A client MUST NOT send header fields in a TRACE request containing sensitive data that might be disclosed by the response.  For example, it would be foolish for a user agent to send stored user credentials [Part7] or cookies [RFC6265] in a TRACE request.  The final recipient SHOULD exclude any request header fields from the response body that are likely to contain sensitive data.

> TRACE allows the client to see what is being received at the other end of the request chain and use that data for testing or diagnostic information.  The value of the Via header field (Section 5.7.1 of [Part1]) is of particular interest, since it acts as a trace of the request chain.  Use of the Max-Forwards header field allows the client to limit the length of the request chain, which is useful for testing a chain of proxies forwarding messages in an infinite loop.

> A client MUST NOT send a message body in a TRACE request.

> Responses to the TRACE method are not cacheable.

### `is_method_options :in`

Return TRUE if the `method :var` is OPTIONS; FALSE otherwise.

### `options :var`

Return a key-value headers and their values.

By default _Allow_ will point to `allowed_methods :var` and _Accept-Patch_ to `patch_content_types_accepted :var`.

### `process_options : in`

Set `options :var` as `Operation.response.headers`.

Return TRUE if succeeded; return FALSE otherwise.

Reference: [HTTPbis](http://tools.ietf.org/html/draft-ietf-httpbis-p2-semantics-22#section-4.3.7), [RFC2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec9.html#sec9.2)

> The OPTIONS method requests information about the communication options available on the request/response chain identified by the effective request URI.  This method allows a client to determine the options and/or requirements associated with a resource, or the capabilities of a server, without implying a resource action.

> [...]

> A server generating a successful response to OPTIONS SHOULD send any header fields that might indicate optional features implemented by the server and applicable to the target resource (e.g., Allow), including potential extensions not defined by this specification.  The response payload, if any, might also describe the communication options in a machine or human-readable representation.  A standard format for such a representation is not defined by this specification, but might be defined by future extensions to HTTP.  A server MUST generate a Content-Length field with a value of "0" if no payload body is to be sent in the response.

> [...]
-->

### `payload_exists : in`

Return TRUE if the request has a payload (`Operation.headers.content-length` greater than 0); return FALSE otherwise.

### `is_payload_too_large :bin`

Return TRUE if the request payload is too large; return FALSE otherwise.

Reference: [HTTPbis](http://tools.ietf.org/html/draft-ietf-httpbis-p2-semantics-22#section-6.5.11), [RFC2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.14)

> The 413 (Payload Too Large) status code indicates that the server is refusing to process a request because the request payload is larger than the server is willing or able to process.  The server MAY close the connection to prevent the client from continuing the request.

> If the condition is temporary, the server SHOULD generate a Retry-After header field to indicate that it is temporary and after what time the client MAY try again.

### `post_content_types_accepted :var`

Return a list of key-value POST content-types and their handlers (i.e. deserializers to `Context.request_entity`).

By default, handle `application/x-www-form-urlencoded`.

### `patch_content_types_accepted :var`

Return a list of key-value PATCH content-types and their handlers (i.e. deserializers to `Context.request_entity`).

### `put_content_types_accepted :var`

Return a list of key-value PUT content-types and their handlers (i.e. deserializers to `Context.request_entity`).

### `content_types_accepted :var`

Return a list of key-value content-types and their handlers (i.e. deserializers to `Context.request_entity`).

By default it will call the callback specific to the request method.

### `is_content_type_accepted :in`

Return TRUE if `Operation.headers.content-type` matches keys of `content_types_accepted :var`; return FALSE otherwise.

The matching must follow specific rules. FIXME

Reference: [HTTPbis](http://tools.ietf.org/html/draft-ietf-httpbis-p2-semantics-22#section-6.5.13), [RFC2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.16)

> The 415 (Unsupported Media Type) status code indicates that the origin server is refusing to service the request because the payload is in a format not supported by the target resource for this method.  The format problem might be due to the request's indicated Content-Type or Content-Encoding, or as a result of inspecting the data directly.

### `content_types_accepted:handler :bin`

Deserialize `Operation.representation` into `Context.request_entity`.

Return TRUE if succeeded; return FALSE otherwise.

### `is_forbidden :bin`

Return TRUE if the semantics of the request (e.g. `Operation.method`, `Context.request_entity`) trigger a forbidden operation; return FALSE otherwise.

Reference: [HTTPbis](http://tools.ietf.org/html/draft-ietf-httpbis-p2-semantics-22#section-6.5.3), [RFC2616](http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html#sec10.4.4)

> The 403 (Forbidden) status code indicates that the server understood the request but refuses to authorize it.  A server that wishes to make public why the request has been forbidden can describe that reason in the response payload (if any).

> If authentication credentials were provided in the request, the server considers them insufficient to grant access.  The client SHOULD NOT repeat the request with the same credentials.  The client MAY repeat the request with new or different credentials.  However, a request might be forbidden for reasons unrelated to the credentials.

> An origin server that wishes to "hide" the current existence of a forbidden target resource MAY instead respond with a status code of 404 (Not Found).

### `is_request_ok :bin`

If you want to validate the request beyond the implemented decisions, this is the place to do it.

Return TRUE if the request looks ok; return FALSE otherwise.