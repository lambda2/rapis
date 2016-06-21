# R.API.S

## REST API Standard

> A specification proposal for Rest API's


### 1. Design
The API must embrace RESTful design principles

Introduction

Enchant is a fully hosted solution to manage email and twitter based customer support. We provide a REST API built on pragmatic RESTful design principles.

Our API uses resource-oriented URLs that leverage built in features of HTTP, like authentication, verbs and response codes. All request and response bodies are JSON encoded, including error responses. Any off-the-shelf HTTP client can be used to communicate with the API.

We believe an API is a user interface for a developer - accordingly, we've made sure our API can be easily explored from the browser!

Changes

This is a versionless API. Advance notification of breaking changes will be available in this document and will be sent by email to our developer mailing list.

### 2. Security

The API must be served over SSL.

### 3. Verbs

|       Verb   |                     Description                       |      Fallback      |
|--------------|------------------------------------------------------:|-------------------:|
|  GET         | Used for retrieving resources.                        |                    |
|  POST        | Used for creating resources.                          |                    |
|  PATCH / PUT | Used for updating resources.                          |                    |
|  DELETE      | Used for deleting resources.                          |                    |

#### Fallback header

The HTTP client that doesn't support PUT, PATCH or DELETE requests must send a POST request with an `X-HTTP-Method-Override` header specifying the desired verb.

The API must correctly handle this header. When it is set, it take precedence over the original request method.

### Status codes

**The API must uses descriptive HTTP response codes to indicate the success or failure of request.**

Codes in the 2xx range must indicate a success, codes in the 4xx range must indicate an error that failed given the information provided (e.g., a required parameter was omitted, a charge failed, etc.), and codes in the 5xx range must indicate a server-side error (e.g., the server is unavailable).

The API must use the following status codes:

|          Http Code        |                               Meaning                                     |
|---------------------------|--------------------------------------------------------------------------:|
| 200 OK                    | Request succeeded. Response included                                      |
| 201 Created               | Resource created. URL to new resource in Location header                  |
| 204 No Content            | Request succeeded, but no response body                                   |
| 400 Bad Request           | Could not parse request                                                   |
| 401 Unauthorized          | No authentication credentials provided or authentication failed           |
| 403 Forbidden             | Authenticated user does not have access                                   |
| 404 Not Found             | Page or resource not found                                                |
| 415 Unsupported Media Type| POST/PUT/PATCH request occurred without a application/json content type   |
| 422 Unprocessable Entry   | A request to modify or create a resource failed due to a validation error |
| 429 Too Many Requests     | Request rejected due to rate limiting                                     |
| 500 Internal Server Error | An internal server error occured                                          |
| 502 Bad Gateway           | The server was acting as a gateway or proxy and received an invalid response from the upstream server |
| 503 Service Unavailable   | The server is currently unable to handle the request.                     |


### Errors


### Parameters


### Custom headers




### Versioning



-> Use [semantic versioning](http://semver.org/)

### Pagination


### Filtering


### Sorting


### Searching


### Embedding


### Counting


### Enveloping



### Sources


- [Principles of good RESTful API Design](https://codeplanet.io/principles-good-restful-api-design/)

- [Semver](http://semver.org/)
