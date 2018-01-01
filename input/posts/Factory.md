Title: "Name: Factory"
Published: 2017-08-21
Tags: REST, HTTP
---
A factory resource is one that is used to create another resource. 
 
## Problem
Sometimes it is difficult to create a complete resource representation on the client without some involvement from the server. It may be that the server is responsible for creating a unique identifier for the new resource making it difficult for the client to do a PUT operation. Or other attributes of the resource are dependent on existing server state.  For example, creating a order resource where the order quantity may be adjusted based on in-stock quantity.

## Solution
The POST method is used to indicate an unsafe request is being made and the response should return a `201 - Created` with a `Location` header that points to the newly created resource. 

```
POST /FooFactory
=>
201 Created 
Location: http://example.org/foo/99 
```

There may or may not be a request body. For some resource factories it may be required to pass some kind of parameters in the request body.  In other cases a request body may just be used to override default values.

The response body may choose to include a representation of the newly created resource, in which case a `Content-Location` header may be appropriate.  Or the response body could contain status information related to the creation request.

It may be useful for the ResourceFactory to implement a GET method that can return a template with default values that can be modified by the client before being POSTed back to the ResourceFactory.

## Discussion
This pattern naturally occurs for most scenarios where `POST` is used to create new resources. Often the factory resource doubles as a collection of the resource to be created. e.g. `POST /messages` might create a resource that appears structurally as a child resource `/messages/75`.  However, there is no requirement for the new resource to be created with a URL that has any correlation to the ResourceFactory.

## Related Patterns
[Sandbox]