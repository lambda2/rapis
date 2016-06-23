# R.API.S

## REST API Standard

A 21th century specification proposal for Rest API's


---------------

#### This specification is intended to establish an agreement on the behavior of an API, no matters the format (json, xml, etc...).


- 1. Design
- 2. Formatting
- 3. Security
- 4. Verbs
  - Fallback header
  - Location header
  - Resource on update and create actions
- 5. Status codes
- 6. Errors
- 7. Parameters
- 8. Custom HTTP headers
- 9. Versioning
- 10. Pagination
- 11. Filtering
- 12. Sorting
- 13. Searching
- 14. Embedding
- 15. Selecting
- 16. Caching
- 17. Asynchronous processing
- Sources

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

> TODO: more ?

### 3. Security

- The API must be served over SSL, using `https`. It must not redirect on non-SSL urls.

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

The server must correctly handle this header. When it is set, it take precedence over the original request method.

#### Location header

When a resource is created (with a `POST` request), the response must contain a `Location` header with the link of the new resource.

#### Resource on update and create actions

When a resource is created or modified (e.g., with a POST, PUT or PATCH request), the response must contain the created or updated respresentation of the resource.

### 5. Status codes

**The API must uses descriptive HTTP response codes to indicate the success or failure of request.**

Codes in the 2xx range must indicate a success, codes in the 4xx range must indicate an error that failed given the information provided (e.g., a required parameter was omitted, a charge failed, etc.), and codes in the 5xx range must indicate a server-side error (e.g., the server is unavailable).

The server must respond with the following status codes, according to the situation:

|          Http Code        |                               Meaning                                     |
|---------------------------|---------------------------------------------------------------------------|
| 200 OK                    | Request succeeded. Response included                                      |
| 201 Created               | Resource created. URL to new resource in Location header                  |
| 204 No Content            | Request succeeded, but no response body                                   |
| 303 See other             | The resource is in another location. See [Asynchronous processing](#asynchronous-processing). |
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


### 6. Errors

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


### 7. Parameters

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
    "created_at": "2016-07-25T12:19:33Z"
  }
}
```


### 8. Custom HTTP headers

All non-standard HTTP headers must begin by a `X-`.

For example, for rate limiting, the `X-Rate-Limit-Limit`, `X-Rate-Limit-Remaining` and `X-Rate-Limit-Reset` headers should be used.

### 9. Versioning

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

### 10. Pagination

Requests for collections should be paginated, and return a limited number of results.

In this case, the client must be able to change the requested page using the `page[number]` parameter.

The client should also be able to change the number of items returned per page using the `page[size]` parameter.

> A lot of services uses the `page` and the `per_page` parameters to set the page number and the number of items per page. The server should be able to support both of theses parameters.

A paginated response should have: 
- A [`Link` header](https://tools.ietf.org/html/rfc5988), which should contain links to the next, previous, first and last resources.
- A `X-Page` header, containing the current page.
- A `X-Per-Page` header, containing the number of items per page.
- A `X-Total` header, containing the total items count.


```HTTP
GET /unicorns?page[size]=2 HTTP/1.1
Content-Type: application/json
Host: api.example.org

HTTP/1.1 200 OK
Content-Type: application/json
Link: <https://api.example.org/unicorns?page[number]=2&page[size]=2>; rel="last", <https://api.example.org/unicorns?page[number]=2&page[size]=2>; rel="next"
X-Page: 1
X-Per-Page: 2
X-Total: 4

[
  {
    "id": 1,
    "name": "Charles",
    "color": "yellow",
    "created_at": "2016-07-25T12:19:33Z"
  },
  {
    "id": 2,
    "name": "Zoe",
    "color": "green",
    "created_at": "2016-07-25T12:19:33Z"
  }
]
```

### 11. Filtering

The client should be able to filter resource collections using the `filter` parameter. In this case, only the fields matching the given filter(s) will be returned.

The value of the `filter` parameter must be a hash of the filter name as a key, and a comma-separated list of the requested values as a value.

```HTTP
GET /unicorns?filter[color]=yellow HTTP/1.1
Content-Type: application/json
Host: api.example.org

HTTP/1.1 200 OK
Content-Type: application/json

[
  {
    "id": 1,
    "color": "yellow"
  },
  {
    "id": 3,
    "color": "yellow"
  }
]
```


### 12. Sorting

The client should be able to sort resource collections according to one or more fields using the `sort` parameter. The value for `sort` must represent sort fields.

Sorting on multiple fields should be done by allowing comma-separated sort fields. In this case, sort fields should be applied in the order specified.

The sort order for each sort field must be ascending unless it is prefixed with a minus (`-`), in which case it must be descending.

```HTTP
GET /unicorns?sort=color,-name HTTP/1.1
Content-Type: application/json
Host: api.example.org

HTTP/1.1 200 OK
Content-Type: application/json

[
  {
    "id": 2,
    "name": "Zoe",
    "color": "green",
    "created_at": "2016-07-25T12:19:33Z"
  },
  {
    "id": 4,
    "name": "John",
    "color": "purple",
    "created_at": "2016-07-25T12:19:33Z"
  },
  {
    "id": 3,
    "name": "Mike",
    "color": "yellow",
    "created_at": "2016-07-25T12:19:33Z"
  },
  {
    "id": 1,
    "name": "Charles",
    "color": "yellow",
    "created_at": "2016-07-25T12:19:33Z"
  }
]
```

If the server does not support sorting as specified in the query parameter `sort`, it must return a `400 Bad Request` status code.

### 13. Searching

The client should be able to search on resource collections fields using the `search` parameter. In this case, only the fields matching the given search(s) will be returned.

The value of the `search` parameter must be a hash of the search field as a key, and the query as a value.

```HTTP
GET /unicorns?search[name]=e HTTP/1.1
Content-Type: application/json
Host: api.example.org

HTTP/1.1 200 OK
Content-Type: application/json

[
  {
    "id": 1,
    "name": "Charles",
    "color": "yellow",
    "created_at": "2016-07-25T12:19:33Z"
  },
  {
    "id": 2,
    "name": "Zoe",
    "color": "green",
    "created_at": "2016-07-25T12:19:33Z"
  },
  {
    "id": 3,
    "name": "Mike",
    "color": "yellow",
    "created_at": "2016-07-25T12:19:33Z"
  }
]
```

A global search on a resource collection should be implemented using directly a value instead of a hash for the `search` parameter.

### 14. Embedding

The client should be able to include data related to (or referenced) from the resource being requested using the `embed` parameter. The value of the `embed` parameter must be a comma separated list of fields to be embedded. Dot-notation must be used to refer to sub-fields.

```HTTP
GET /unicorns?embed=country.name HTTP/1.1
Content-Type: application/json
Host: api.example.org

HTTP/1.1 200 OK
Content-Type: application/json

[
  {
    "id": 1,
    "name": "Charles",
    "color": "yellow",
    "created_at": "2016-07-25T12:19:33Z",
    "country": {
      "name": "Australia"
    }
  },
  {
    "id": 2,
    "name": "Zoe",
    "color": "green",
    "created_at": "2016-07-25T12:19:33Z",
    "country": {
      "name": "Italy"
    }
  },
  {
    "id": 3,
    "name": "Mike",
    "color": "yellow",
    "created_at": "2016-07-25T12:19:33Z",
    "country": {
      "name": "U.S.A"
    }
  },
  {
    "id": 4,
    "name": "John",
    "color": "purple",
    "created_at": "2016-07-25T12:19:33Z",
    "country": {
      "name": "France"
    }
  }
]
```

### 15. Selecting

The client should be able to select only specific fields in the response using the `fields` parameter. In this case, only the requested fields will be returned.

The value of the `fields` parameter must be a hash of the resource name as a key, and a comma-separated list of the fields names to be returned as a value.

```HTTP
GET /unicorns?fields[unicorns]=id,color HTTP/1.1
Content-Type: application/json
Host: api.example.org

HTTP/1.1 200 OK
Content-Type: application/json

[
  {
    "id": 1,
    "color": "yellow"
  },
  {
    "id": 2,
    "color": "green"
  },
  {
    "id": 3,
    "color": "yellow"
  },
  {
    "id": 4,
    "color": "purple"
  }
]
```

If the server does not support selection as specified in the query parameter `fields`, it must return a `400 Bad Request` status code.


### 16. Caching

Server should generate a [ETag header](http://en.wikipedia.org/wiki/HTTP_ETag) containing a hash or checksum of the representation. This value should change whenever the output representation changes.

### 17. Asynchronous processing

When a resource creation or update is asynchronously processed, the request should return a `202 Accepted` status code with a link in the `Content-Location` header which should redirect to the resource when the job processing is done.


```HTTP
POST /movies HTTP/1.1
Accept: application/json
Content-Type: application/json
Host: api.example.com

{
  "movie": {
    "name": "Charlie the Unicorn",
    "source": "https://www.youtube.com/watch?v=CsGYh8AacgY"
  }
}

HTTP/1.1 202 Accepted
Content-Type: application/json
Content-Location: https://api.example.com/movies/jobs/42

{}
```

When the job process is done, the request of the url in the `Content-Location` header should return a `303 See other` status code with the created resource link in `Location` header.

```HTTP
GET /movies/jobs/42 HTTP/1.1
Accept: application/json

HTTP/1.1 303 See other
Content-Type: application/json
Location: https://api.example.com/movies/3
```

### Sources

- [Nouns are good, verbs are bad](http://apigee.com/about/blog/technology/restful-api-design-nouns-are-good-verbs-are-bad)
- [Principles of good RESTful API Design](https://codeplanet.io/principles-good-restful-api-design/)
- [Best Practices for Designing a Pragmatic RESTful API](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api#useful-post-responses)
- [Semver](http://semver.org/)
- [JSON API specification](http://jsonapi.org/)
