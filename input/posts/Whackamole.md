Title: "Name: Whack-a-mole"
Published: 2017-08-21
Tags: REST, HTTP
---
A type of resource that when deleted, re-appears as a different resource.

## Problem
A user wishes to remove, cancel, make inactive, or archive an entity, the system wishes to maintain a history that the entity did exist at some point in time and also make that available as a resource.

## Solution
Implement the DELETE method on the resource that is to be affected.  The implementation should delete the existing resource as requested and create a new resource to represent the affected resource.

    DELETE /order/234243 HTTP/1.1
    Host: example.org
    =>
    HTTP/1.1 204 No Content
    Location: http://example.org/cancelled-orders/234243

## Discussion
There are numerous alternative approaches to solving this problem. Internally, this change is usually implemented by changing some status field.  Both the  `Miniput` pattern and the `Bucket` pattern can be used to make status changes.  However, there are some advantages of using the DELETE method to perform the operation.  If the resource allows caching, then the DELETE method can be used as a signal to intermediary caches to flush the cached copy.  Using the DELETE method infers that the status change is likely to be a one-way change. It conveys the message that to the consumer that reversing the DELETE operation many be difficult or impossible.

In the example, there is a Location header that indicates where the new resource has been created.  This is not required for this pattern but it may be useful for some clients.

The content that is made available by the secondary resource is not required to be a duplicate of the primary resource. The secondary resource may only contain a subset of the original information, or additional information not in the primary resource.

## Related Patterns
Miniput, Bucket
