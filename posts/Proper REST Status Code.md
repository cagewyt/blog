
Finding the proper REST response code can be painful sometimes when designing REST API, because

1.  The list of HTTP status code is (unnecessary) long (75 for now on [wiki](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)).
2.  Too many possible status codes for a REST API make the API harder to be consumed by clients, whereas too few status codes do not provide enough granularity for errors.

We all use 200 OK, but how often do we use the following?

-   205 Reset Content
-   300 Multiple Choices
-   419 Authentication Timeout

This article discusses what is the proper code subset that are frequently used and well-understood, and still provide enough granularity of errors.

Steve Marx from Dropbox has a nice article talking about "[How many HTTP status codes should your API use?](https://blogs.dropbox.com/developers/2015/04/how-many-http-status-codes-should-your-api-use/)". In short, the Twitter documents [15 status codes](https://dev.twitter.com/overview/api/response-codes). Dropbox uses [8 specific status codes](https://www.dropbox.com/developers/core/docs#error-handling). For error code, it is common to provide a detailed error message or specific error code in JSON response.

Before I begin, here are some nice sites about this topic.

-   [REST API tutorial-HTTP status code](http://www.restapitutorial.com/httpstatuscodes.html)
-   [REST patterns-HTTP status code](http://www.restpatterns.org/HTTP_Status_Codes)
-   HTTP Status Codes and [RESTful API crafting](http://blog.thefrontiergroup.com.au/2012/08/http-status-codes-and-restful-api-crafting/) from The Frontier Group Journal with interesting discussion.

Here is a subset of REST status code that I find is proper for designing REST API.

### (2xx) Success Code

-   **200 OK:** success. Any supplementary message can be provided via JSON response, such as the URI of the affected resource.
-   **201 Created:** for creating a new resource.
-   **202 Accepted:** to acknowledge that the user's request is accepted. A task in the backend may be started, which may be a time-consuming. For example, a request to start generating annual finance report. Once done, a notification message is sent to the user. This is different from the case when the result of a request is immediately available and 200 OK is returned.
-   **204 No Content:** There is nothing to see here. Useful if you DELETE an object successfully. Personally, I don't see any advantage using this.

### (4xx) Client Error Codes

-   **400 Bad Request:** Nothing to say here.
-   **401 Unauthorized:** The 'Unauthorized' is confusing. 401 semantically means ["unauthenticated"](https://en.wikipedia.org/wiki/Authentication "Authentication"). Authentication failure, the request does not have valid credentials.
-   **403 Forbidden:** Authorization failure, the user is not allowed/authorized to perform the action.
-   **404 Not Found:** The URI resource is not found, maybe it does not exist.
-   **405 Method Not Allowed:** Used to indicate that the requested URI exists, but the requested HTTP method is not applicable. For example, POST _/users/12345_ where the API doesn't support creation of resources this way (with a provided ID). The Allow HTTP header must be set when returning a 405 to indicate the HTTP methods that are supported. In the previous case, the header would look like "Allow: GET, PUT, DELETE". From [Manuel Boy](https://phraseapp.com/blog/posts/best-practice-10-design-tips-for-apis/).
-   **409 Conflict:** when a resource conflict would be caused by the request. Duplicate entries, such as trying to create two users with the same information. Mostly, a duplicate in the database.
-   **410 Gone:** for URI has been permanently (and intentionally) deleted. This informs the client to remove any references to the resource.

### (5xx) Server Error Codes

-   **500 Interval Server Error:** Never return this intentionally. The general catch-all error when the server-side throws an exception. Usually reply on the server side framework to catch exception and convert to 500 with proper error response.
-   **501 Not Implemented:** The server does not support the functionality required to fulfill the request. This is the appropriate response when the server does not recognize the request method and is not capable of supporting it for any resource.
-   **503 Service Unavailable:** The server is currently unable to handle the request due to a temporary overloading or maintenance of the server.

**Tips:**

-   Log ERROR for 5xx error (with exception stack trace), log debug/warning for 4xx error. On the server side, one may attempt to log any error code, which makes log file extremely space expensive. If this is your concern, minimized the log of 4xx errors because it is not the server's fault.
-   Your web server and REST framework/lib may already generate some status codes for you, such as 503 Service Unavailable when server is down, 400 Bad Request when payload has syntax error, 404 Not Found when URI is not defined, 405 Method Not Allowed when the operation is not defined, etc.

### Status Code for Validation Error:

This is a long debated topic on stackoverflow: [link 1](http://stackoverflow.com/questions/3290182/rest-http-status-codes-for-failed-validation-or-invalid-duplicate), [link2](http://stackoverflow.com/questions/1959947/whats-an-appropriate-http-status-code-to-return-by-a-rest-api-service-for-a-val)

Short answer: If you want minimal number of generic code, then go 400 with JSON message. Don't bother arguing about 'malformed syntax'.

-   [RFC2616](https://www.ietf.org/rfc/rfc2616.txt): 400 means: The request could not be understood by the server due to malformed syntax.
-   [RFC 7231](http://tools.ietf.org/html/rfc7231): 400 means: the server cannot or will not process the request due to something that is perceived to be a client error.

The new standard supersedes the previous one, so 400 is ok for validation error, although quite generic.

If you are willing to use more specific codes, you have the following options:

-   **[422 Unprocessable Entity:](http://greenbytes.de/tech/webdav/rfc4918.html#rfc.section.11.2)** seems getting more and more popular, used by Rails out of the box.
-   **409 Conflict:** for duplicate
-   **403 Forbidden:** 422 makes more sense for this.

Last, fun to read: [418 I'm a teapot (RFC 2324)](https://sitesdoneright.com/blog/2013/03/what-is-418-im-a-teapot-status-code-error).
