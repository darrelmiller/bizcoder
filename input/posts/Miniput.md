Title: "Name: Miniput"
Published: 2017-08-21
Tags: REST, HTTP
---
A resource designed to enable doing a partial updates to another resource.

## Problem
It is frequently desirable to allow a client to update a subset of information contained in a resource. The HTTP specification does not allow the PUT method to perform partial updates to a resource. The PATCH method can be used to make an incremental change to a resource, but it requires a specially formatted patch format payload and PATCH is not guaranteed to be idempotent.

## Solution
Creating "child" resources to represent fragments of another resource allows partial updates of the "parent" resource by doing complete updates of the "child" resource.

Assuming we have a "parent" resource that looks like this,

    GET /Contact/2323
    Host: example.org
    =>
    HTTP/1.1 200 OK
    Content-Type: application/json
    Content-Length: ...

    {
        "firstname" : "James",
        "lastname" : "Bond",
        "telephone" : ""
    }

We can update the telephone number of the contact by doing a PUT to a "child" resource.

    PUT /Contact/2323/telephone
    Host: example.org
    Content-Type: text/plain
    Content-Length: 12

    514-426-0102
    =>
    HTTP/1.1 204 No Content

Retreiving the "parent" resource would give the following,

    GET /Contant/2323
    Host: example.org
    =>
    HTTP/1.1 200 OK
    Content-Type: application/json
    Content-Length: ...

    {
        "firstname" : "James",
        "lastname" : "Bond",
        "telephone" : "514-426-0102"
    }

If it is preferable to return the updated representation from the PUT, it can be done as follows,

    PUT /Contact/2323/telephone
    Host: example.org
    Content-Type: text/plain
    Content-Length: 12

    514-426-0102
    =>
    HTTP/1.1 200 OK
    Content-Location: http://example.org/Contact/2323
    Content-Type: application/json
    Content-Length: ...

    {
        "firstname" : "James",
        "lastname" : "Bond",
        "telephone" : "514-426-0102"
    }

The `Content-Location` header is required in this case because the returned representation does not come from the resource that is the target of the PUT.

## Discussion
This solution is effective for making updates to a limited number of fragments of a resource.  However it is less than ideal for a general purpose way of doing partial updates as it requires the creation of a resource for all combinations of possible partial updates.

As PUT is used to make these changes, the operation is idempotent and unsafe.  One risk of using this approach is that cache invalidation becomes more tricky.  HTTP caches will naturally invalidate representations when they see an inbound PUT on a resource.  However, because a `Miniput` is actually updating the "parent" different resource, the cache will not know to invalid the "parent" resource.  This issue can be addressed using [cache-invalidation link headers](http://tools.ietf.org/html/draft-nottingham-linked-cache-inv-04) for caches that support that technique.

One useful characteristic of this approach is that "child" resources can use a very simple media type for performing the update.  Frequently the `text/plain` media type is suffient for partial updates that only update a single property. 

An additional benefit of creating these very finely grained resources for doing updates is that they can also be used for retrieving just a single piece of information about a resource.  However, these benefits can quickly be lost if there are multiple pieces of information required and a client starts making multiple round-trips to retrieve that information.

## Related
[Bucket](), [Whack-a-mole]()

