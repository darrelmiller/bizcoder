Title: "Name: Progress"
Published: 2017-08-21
Tags: REST, HTTP
---
A progress resource is usually a temporary resource that is created automatically by the server to provide status on some long running process that has been initiated by a client. It is used to provide feedback to an end user and point to the results of an operation once it has completed.

## Problem
HTTP is a client/server request and response protocol.  It does not provide for server initiated communications.  If a client makes a request to a server that is likely to take a long time, e.g. 5 seconds or longer, it may be problematic to maintain an open connection to the client whilst the operation completes.  HTTP allows a server to return a 202 response to indicate the server has a started the operation, but the client may still wish to monitor the progress of the operation and inspect the final results.

## Solution
In order to communicate to the client that a request has been successfully made but the response is not yet ready, the server returns the 202 status code. Additionally it can return a Location header that has a URL which points to a `Progress` resource. 

```
POST /deploy-vm HTTP/1.1
Host: http://example.org
=>
202 Accepted
Location: http://example.org/deployprocess/10102
```

There is no standard for progress representations however, they generally should contain information about the current state of the long running operation and may contain a link to cancel the operation, or perform other useful operations.  An example of what a progress representation might look like is:  

```
<status state="busy" progress="2/75" message="This is a message"/>
```

This example is taken from an [experimental media type](https://github.com/tavis-software/Tavis.Status) `application/status+xml`.

When the long running process does complete and there is a resource that contains the results of the operation, then the progress resource should contain a link that points to the results. 

```
{ "status" : {
        "state" : "ok",
        "message" : "Finished generating report",
        "links" : {
            "related" : { "href" : "http://example.org/report/99/output"}
        }
    }
}
```

## Discussion
One challenge of progress resources is knowing how long they should be kept around for.  It is impossible to know when the client will query for the result of the operation.  In some cases having a permanent record of the invocation may be useful.  Other cases it may be preferred to ask the client to clean up the progress resource using a `DELETE` method once it has finished with it.  It may also be reasonable to set an expiry time on the resource and have a cleanup process remove expired progress resources.  If the expiry option is chosen, and a HTTP cache sits in front of the origin server, then setting a Cache-Control max-age value can be used to make the resource available for a limited amount of time.    

As mentioned earlier it is possible for the progress resource to advertize other useful services using links.  Examples include:

 - Cancel link
 - Webhook registration link
 - Long polling link
 - Web sockets link

Including these kind of links is obviously completely optional but if the links are identified using some standard identifier, then clients that support these types of features can choose to take advantage of them or not based on their availabilty.  Clients could prompt a user and let them choose how they wish to be notified of completion.

When building APIs, developers usually pre-define which resources will be processed asynchronously, and which will be synchronous.  A client developer has prior knowledge of the expected interaction pattern.  However, it doesn't necessarily need to be designed this way.  Using the `Prefer` header with the `async` parameter a client can make a request and declare to the server a preference for how it would like the result to be processed.  If the server can process the request asynchronously then it will, otherwise it can ignore the request.  The client needs to be prepared to deal with either a 200 or a 202 response.

This capability can be useful for supporting users who issue queries that based on the parameter values may take significantly varying amounts of time.  An end user could be asked if they want to wait for the results or be notified when ready.  Also, if it is desireable to support some clients that can do async and some that cannot, a server could be informed of the client's capabilities via the `prefer` header.  

One particularly interesting challenge is when accessing the progress resource fails.  If it is a transient failure, and future requests fail, then there is no issue.  However, if the progress resource cannot be accessed, then the client does not know if the long running resource succeeded.  If long running operation was initiated by a safe or idempotent request then the requires can be reissued.  However, if the request was initiated by a `POST` then there needs to be some fallback method for the client to deterime if the earlier request succeeded or failed.

## Related Patterns
