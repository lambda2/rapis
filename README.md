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

### Security

### Verbs

VERB 
DESCRIPTION 
GET 
Used for retrieving resources. 
POST 
Used for creating resources. 
PUT 
Used for changing/replacing resources or collections. 
DELETE 
Used for deleting resources. 

### Status codes


Errors
----------------

The 42 API uses the following error codes:

| Http Code | Error code | Meaning |
|-----------|-----------:|--------:|
| 400       |            | The request is malformed |
| 401       |            | Unauthorized |
| 403       |            | Forbidden|
| 404       |            | Page or resource is not found|
| 422       |            | Unprocessable entity|
| 500       |            | We have a problem with our server. Please try again later.|
| Connection refused       |            |Most likely cause is not using HTTPS. |


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
