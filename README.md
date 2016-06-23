# R.API.S

## REST API Standard

A 21th century specification proposal for Rest API's


### 1. Design

The API must embrace RESTful design principles. It must be resource-based, and each resource representation must contain enough information to modify or delete the resource on the server, provided it has permission to do so.

- The resources names and fields must be [snake_case](http://en.wikipedia.org/wiki/Snake_case).

> [snake_case is 20% easier to read than camelCase](http://ieeexplore.ieee.org/xpl/articleDetails.jsp?tp=&arnumber=5521745). That impact on readability would affect API explorability and examples in documentation. 

- The resources names must be nouns.

> [Nouns are good, verbs are bad](http://apigee.com/about/blog/technology/restful-api-design-nouns-are-good-verbs-are-bad)

- The endpoints names must be plural.

> You don't want to deal with complex pluralization (e.g., foot/feet, child/children, people/people). Keep it simple.

### 2. Formatting

- Dates must be returned in ISO 8601 format (YYYY-MM-DDTHH:MM:SSZ).
- Geographic coordinates must be returned in `[-]d.d, [-]d.d` format (e.g., 12.3456, -98.7654).

### 3. Security

The API must be served over SSL, using `https`. It must not redirect on non-SSL urls.

> Always using SSL guaranteed encrypted communications, and allow use of simple access tokens.

### 4. Verbs

|       Verb   |                     Description                       |
|--------------|-------------------------------------------------------|
|  GET         | Used for retrieving resources.                        |
|  POST        | Used for creating resources.                          |
|  PATCH / PUT | Used for updating resources.                          |
|  DELETE      | Used for deleting resources.                          |

#### Fallback header

The HTTP client that doesn't support PUT, PATCH or DELETE requests must send a POST request with an `X-HTTP-Method-Override` header specifying the desired verb.

The API must correctly handle this header. When it is set, it take precedence over the original request method.

### Status codes

**The API must uses descriptive HTTP response codes to indicate the success or failure of request.**

Codes in the 2xx range must indicate a success, codes in the 4xx range must indicate an error that failed given the information provided (e.g., a required parameter was omitted, a charge failed, etc.), and codes in the 5xx range must indicate a server-side error (e.g., the server is unavailable).

The API must use the following status codes:

|          Http Code        |                               Meaning                                     |
|---------------------------|---------------------------------------------------------------------------|
| 200 OK                    | Request succeeded. Response included                                      |
| 201 Created               | Resource created. URL to new resource in Location header                  |
| 204 No Content            | Request succeeded, but no response body                                   |
| 304 Not Modified          | The response is not modified since the last call. Returned by the cache.  |
| 400 Bad Request           | Could not parse request                                                   |
| 401 Unauthorized          | No authentication credentials provided or authentication failed           |
| 403 Forbidden             | Authenticated user does not have access                                   |
| 404 Not Found             | Page or resource not found                                                |
| 405 Method Not Allowed    | The request HTTP method is not allowed for the authenticated user         |
| 410 Gone                  | The endpoint is no longer available. Usefull for old API versions         |
| 415 Unsupported Media Type| POST/PUT/PATCH request occurred without a application/json content type   |
| 422 Unprocessable Entry   | A request to modify or create a resource failed due to a validation error |
| 429 Too Many Requests     | Request rejected due to rate limiting                                     |
| 500 Internal Server Error | An internal server error occured                                          |
| 502 Bad Gateway           | The server was acting as a gateway or proxy and received an invalid response from the upstream server |
| 503 Service Unavailable   | The server is currently unable to handle the request.                     |


### Errors

- All errors in the 4xx must return a body containing a `error` key, containing the error code.

- This code must be human readable, and identical over the same kinds of errors.

- The body should also contain a `message` key, containing a more detailled description of the error. 

```HTTP
GET /unicorns/4 HTTP/1.1

HTTP/1.1 404 Not Found
Content-Type: application/json

{
   "error": "Not Found",
   "message": "Unable to found unicorn with id '4'"
}
```

- On validation errors (with a 422 Unprocessable Entity status code), the body should contain a `messages` array containing all the validation errors.

```HTTP
POST /unicorns HTTP/1.1

{
  "unicorn": {
    "color": "purple"
  }
}

HTTP/1.1 422 Unprocessable Entity
Content-Type: application/json

{
   "error": "Validation failed",
   "messages": ["name cannot be blank"]
}
```


### Parameters

Resource creation or update parameters must be wrapped in an object as the singular name of the resource.

```HTTP
POST /unicorns HTTP/1.1
Accept: application/json
Content-Type: application/json
Host: api.example.com

{
  "unicorn": {
    "name": "John",
    "color": "purple",
    "country_id": 1
  }
}

HTTP/1.1 201 Created
Content-Type: application/json

{
  "unicorn": {
    "id": 4,
    "name": "John",
    "color": "purple",
    "created_at": "2016-07-25T12:19:33Z",
    "country": {
      "id": 1,
      "name": "France"
    }
  }
}
```


### Custom HTTP headers

All non-standard HTTP headers must begin by a `X-`.

For example, for rate limiting, the `X-Rate-Limit-Limit`, `X-Rate-Limit-Remaining` and `X-Rate-Limit-Reset` headers should be used.

### Versioning

The API must be versioned, and must not have breaking changes whitout version change.

- The client must be able to set the requested version trough the `Accept` header. (e.g., `Accept: application/vnd.myapp.v2+json`).
- Without `Accept` header, the API must use the last stable version.
- The response header must contain a `X-Version` field containing the version used for this request.
- The version field should be formated using the [semantic versioning](http://semver.org/).

```HTTP
GET /unicorns HTTP/1.1
Accept: application/vnd.example-app.v3.1+json
Content-Type: application/json
Host: api.example.org

HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 477
Content-Type: application/json
X-Version: 3.1

{
  [...]
}
```

### Pagination


### Filtering


### Sorting

The client should be able to 

### Searching


### Embedding


### Selecting

The client should be able to select only specific fields in the response using the `fields` parameter. In this case, only the requested fields will be returned.

The value of the `fields` parameter must be a hash of the resource name as a key, and a comma-separated list of the fields names to be returned as a value.

```HTTP
GET /unicorns?fields[unicorns]=id,color&fields[countries]=name HTTP/1.1
Content-Type: application/json
Host: api.example.org

HTTP/1.1 200 OK
Content-Type: application/json

[
  {
    "id": 1,
    "color": "yellow",
    "country": {
      "name": "Australia"
    }
  },
  {
    "id": 2,
    "color": "green",
    "country": {
      "name": "Italy"
    }
  },
  {
    "id": 3,
    "color": "red",
    "country": {
      "name": "U.S.A"
    }
  },
  {
    "id": 4,
    "color": "purple",
    "country": {
      "name": "France"
    }
  }
]
```


### Counting


### Enveloping


### Caching


### Sources

- [Nouns are good, verbs are bad](http://apigee.com/about/blog/technology/restful-api-design-nouns-are-good-verbs-are-bad)
- [Principles of good RESTful API Design](https://codeplanet.io/principles-good-restful-api-design/)
- [Best Practices for Designing a Pragmatic RESTful API](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api#useful-post-responses)
- [Semver](http://semver.org/)
- [JSON API specification](http://jsonapi.org/)
