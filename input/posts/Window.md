Title: "Name: Window"
Published: 2017-08-21
Tags: REST, HTTP
---

A resource that provides access to a subset of a larger set of information through the use of parameters that filter, project and zoom information from the complete set.

## Problem
A server has a set of data that it wishes to enable clients to access.  However, clients don't want to access the entire set of data at once.  Either for performance reasons, or just relevance reasons they only wish to see a subset.  A server needs to design a parameterized URI template that allows the client to specify just the subset that it desires.

Choosing the optimum window size for the client requires accounting for a number of factors:
- latency
- wasted data transfer
- cache variance
- query partitionability

For high latency interactions, the benefits of a smaller windows size must offset the cost of additional round-trips to transfer all of the desired data.
Large window size can cause clients to retrieve more data than required.  This can increase latency, reduce client responsiveness, and on metered networks may actually have a tangible financial impact.
Larger window sizes containing volatile data may become stale more frequently reducing cache hit ratio.
Windows with boundaries based on domain values that are static often cost less to materialize due to backend indexes and suffer less consistency issues.

There are two primary reasons why clients wish to have windowed resources, relevance selection and streaming.  Sometimes a client wants a combination of the two.

### Relevance
When a client only wants to process a specific subset of the available information, it can use a window with the appropriate dimensions and location to retreive the subset. 

   GET /restaurants?country=USA&state=SC&limit=100

Sometimes, an initial subset can be further refined based on category values contained in the results

   GET /restaurants?country=USA&state=SC&style=BBQ

### Streaming
Windows can also be used to create an application level mechanism that simulates streaming.  This enables clients to minimize memory usage on the client as it can process a single window, free related resources and then process the subsequent window.  Sometimes this can be used while searching for a specific piece of information where only the client can evaluate a match.  By retrieving the set of information one window at a time, the client can stop retrieval once the desired information is located.  


## Solution
windows have dimensions that define their contents

natural windows get better cache hits
natural windows have better concurrency characteristics
natural windows often have better performance

Window locations that are defined by domain values are the most useful for clients to select their desired subset.  Domain values have the disadvantage of creating  windows with uneven sizes as they are dependent on the distribution of actual data values.   


synthetic windows can be implemented with reuseable generic code
synthetic windows have a natural order


Paging

limiting via partitioning
window position and size

natural windows will usually be faster than synthetic windows
synthetic windows can have a fixed size 
natural windows can have a max size.

natural windows vs synthentic windows
max window size.  why limit? Trust? 
client memory/cpu
client processing latency
client data transfer cost
server bandwidth
server processing time
server data query time
network transfer time

if you hit a natural window max, suggest natural windows that are subsets.  
default window?

rows is not enough.  it should be by bytes
size metrics

iterating over enforced synthetic windows is worst case scenario



## Discussion
low latency makes finer grained requests more feasible
reducing round trips may increase wasted data transfer

Don't confuse chunking/streaming with windows
if you must use synthetic windows make them as large as possible.  Use quota to manage scale

It is worth noting that using HTTP chunked encoding can be significantly more efficient than using windows to stream a large resultset. However, it does require that client parsing libraries are able to deliver partial results to client applications.

## Related Patterns
Often referred to as `Paging`