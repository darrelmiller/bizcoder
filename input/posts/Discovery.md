Title: "Name: Discovery"
Published: 2017-08-21
Tags: REST, HTTP
---
This type of resource is used to provide a client with the information it needs to be able to access other resources.  This removes the need for a client to hardcode resource URLs which include information such as the protocol scheme, host, path and query string.

## Problem
The primary problem addressed by a discovery resource is allowing API developers to **change various elements of their API without having to notify client developers** of the change and potentially force those customers to redeploy their application.  

A secondary problem that is diminished with the use of a discovery resource relates to use of URL templates for communicating URL patterns to the client. Using **templates eliminates some of the complexity in constructing URLs on the client**. URLs appear to be a simple syntax but they have some tricky encoding rules that help to improve global interoperability. Ad-hoc client side URL construction code often over-simplifies and ignores certain edge cases.

A discovery document can also be used to learn more about how API interactions should occur, in an automated manner, and enable **generic client-side libraries** to simplify client application API interaction code.

The use of a discovery document to identify a set of entry point resource URLs allows a client application to be **pointed to different instances of a service** that have different URLs.  This could be for implementing different staging environments, multi-tenant scenarios, or geo-distributed resources.

## Solution
In order to provide a discovery resource, a client must have two things:  It must have prior knowledge of the location of the discovery resource for a particular service instance and it must know how to intepret the representation that is returned in order to discover other resources. 

Without a higher order service discovery service, a client application must have hard-coded knowledge of the location of the discovery resource.  An approriate place to put this is at the root of an API. For example at `https://api.example.org/`

For some APIs it may be useful to split the discovery resource into multiple discovery resources. The root discovery document should enable discovering the additional discovery resources.  Consider resource volatility and caching when deciding if it is necessary to break down a discovery resource.

Knowledge of how to interpret the discovery document may be implicit, based on a pre-arranged understanding of the contents returned by the discovery resource. An example of this would be the Github discovery document.  

```
GET https://api.github.com  HTTP/1.1
Host: api.github.com

HTTP/1.1 200 OK
Server: GitHub.com
Content-Type: application/json; charset=utf-8
Content-Length: 1942

{
  "current_user_url": "https://api.github.com/user",
  "current_user_authorizations_html_url": "https://github.com/settings/connections/applications{/client_id}",
  "authorizations_url": "https://api.github.com/authorizations",
  "code_search_url": "https://api.github.com/search/code?q={query}{&page,per_page,sort,order}",
  "emails_url": "https://api.github.com/user/emails",
  "emojis_url": "https://api.github.com/emojis",
 ...
```

Alternatively, the understanding of the discovery resource may be explicitly understood based on the media type identifier specified in the `Content-Type` header.

## Discussion
A common objection to the use of a discovery document is that there is always an additional roundtrip when first accessing the API to retrieve the discovery document in order to identify the URL of the target resource. This additional cost can be mitigated by using client side caching tooling. The discovery document is one that is likely to have a fairly long lifespan and with a persistant client cache, the addition cost can be extremely minimal.

Discovery documents are intended to be consumed at run-time by clients.  One example of a document format designed to represent this type of resource is json-home.  There are other documents, often called API description, documents that perform a similar role but are intended to be consumed by tooling at design time.  This class of document includes OpenAPI, API Blueprint, RAML and WADL.  It is feasible that these documents could be repurposed to work for driving a client at runtime also. 

## Related Patterns
Alias