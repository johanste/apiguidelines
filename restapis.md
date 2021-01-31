# Azure REST APIs
The Microsoft REST API guidelines provides general guidelines (requirements) for the design of REST APIs across Microsoft. That there are several "styles" of APIs (e.g. OData, REST, RPC), and the general guidelines provides options for each.

This document is intended to provide a normative description and recommendations specifically for Azure REST APIs. It is intentionally more prescriptive than the general guidelines.

## TLDR

### Request Methods
- **DO** use the correct HTTP verbs for your operation.
- **DO** use PUT against the canonical path of a resource in order to create or replace it if the name of the resource is known a priori.
- **DO** use POST against the canonical path to the collection to create an item in the identity (name) of the resource to create is not known a priori.
- **DO** support updates of all mutable properties of resources using PATCH.
- **DO** support JSON merge-patch as the content type for PATCH operations against resources. You **MAY** also support JSON patch.
- **DO** document if your POST or PATCH operation is idempotent. See also: the [`x-ms-idempotent`](#traits_idempotent) trait. TODO: Add support for new extension

### Response Status Codes
- **DO** use the correct HTTP status codes for your responses.
- **DO** document your top level error codes and under what circumstances they can occur. 

### Resources
- **DO** use the same model as request body and response content for any given URL.
- **DO NOT** use semantically meaningful `null`s in your stored resources. That is, do not distinguish between a null value for a description of a resource and not having a description for the resource.
- **DO** use maps for collections of named resources rather than arrays of objects.
- **DO** use arrays only for collections where order is relevant *or* where the items in the collection does not have a name/identifier.
- **DO** document which properties can be updated after a resource has been created.
- **DO** model child resources as collections of the parent resource when the relationship is a composition.
- **DO** allow direct references of child resources for composite resources.

### Conditional Requests
- **DO** support conditional requests (`if-match`, `if-none-match` and `if-not-modified` headers) for your API. See [optimistic concurrency](#restapi_optimistic_concurrency) for more information.
- **DO** support ETags for your resources.
- **DO** use a stable checksum of the resource representation as the `ETag` value. Do *not* use a random value or an implicit last modified time (or derivation thereof) as the value.

### Versions
- **DO** use query string version parameters unless your family of services has overridden this. 
- **DO** use date-based API versions.
- **DO** append a -preview suffix for API versions that are in preview.

### Service
- **DO** document what property changes will cause service interruption/downtime (e.g. compute resource will be rebooted)


## How to design your API

### Describe your resources.
REST APIs are, by definition, resource centric. Resources represent the nouns in your system.

> The line between nouns and verbs can sometimes be blurry - see [actions](#resource_actions) below...

Since users of your service will be interacting with your resources, it is important that they map to the user's conceptual model of the service. A good litmus test is that you should be able to use the name of the resources in the blog post that introduces your service to your users.

#### Canonical URLs of resources

- Collections of resources

#### Relationships between resources

- Composite resources
- Referenced resources

A **composite resource** is one where the lifetime of a child resource is completely dependent on the lifetime of the parent resource. In other words, a child can never exist absent a parent.

A **referenced resource** is one where the resource can exist independently of any resources referencing it.

```http
GET /directory/{directoryName}/files

PUT /directory/directoryName/files/{fileName}

{
    "permissions": "0x0777"
    ...
}


```

Issue; composite resources and updates using PUT. 
Issue; deleting relationship between objects vs. deleting the target object.
Issue; usage of `link` objects

### <a name="resource_actions"></a> Describe actions on your resources.
Most services go beyond simple CRUD operations on individual resources. 

- Named updates
- Orchestrated changes
- Long running changes

> TODO - insert example. 

The english word _request_ is both a noun and a verb. 

> Does your operation take a long time to complete? Does it keep state? Would someone ask what the current status is? Who invoked the operation? Would someone ever want to cancel the operation?


### Describe your error cases.

## Support normal HTTP semantics

Much of HTTP semantics are nicely described in [RFC 7231](https://tools.ietf.org/html/rfc7231).  Briefly, each HTTP message is either a request sent from a client regarding a resource and an action to be applied in the context of that resource, or a response sent from a server regarding the result of its attempt to parse and execute the requested action.  

A request consists of a resource identifier or URI, a request method, request headers, and sometimes a payload representing the resource or desired action.  

A response consists of a response status code, response headers, and sometimes a payload representing the resource or the result of an action applied in the context of that resource.

It is worth noting the similarity of HTTP to an object oriented programming model, in that there are objects or resources (nouns) and actions or request methods (verbs) that are applied to them.  One notable difference, however, is that a design goal of HTTP is to separate resource identification from request semantics.  As a result, a resource is identified separately from the resource method -- one of a fixed set of verbs that can be applied to the resource.

### 1. Use the correct verbs
Commonly used request methods include:
* [GET](https://tools.ietf.org/html/rfc7231#section-4.3.1) - requests the transfer of a current representation for the target resource.  Corresponds to Read in the CRUD operations.
* [HEAD](https://tools.ietf.org/html/rfc7231#section-4.3.2) - identical to GET, except server must not send a message body in the response.  Can be used to determine if a resource has been modified since the client last accessed it.
* [PUT](https://tools.ietf.org/html/rfc7231#section-4.3.4) - requests that a target resource be created or replaced with the representation in the payload.  Corresponse to Create in the CRUD operations, or Update with replace semantics.
* [PATCH](https://tools.ietf.org/html/rfc5789#section-2) - requests that a set of changes in the message body be applied to the target resource.  Corresponds to Update in the CRUD operations, with semantics specific to the JSON content type.
* [DELETE](https://tools.ietf.org/html/rfc7231#section-4.3.5) - requests that the server remove the target resource.  Corresponds to Delete in the CRUD operations.
* [POST](https://tools.ietf.org/html/rfc7231#section-4.3.3) - requests that the target resource process the payload according to the specific semantics of that resource.  Allows functionality beyond the CRUD operations.

### 2. Status codes
### 3. Headers
[TODO: which to call out?  Separate into request and response headers?  What "stories" to highlight here?]
### 4. Optimistic concurrency using etags/if-match

## Cookbook - API patterns

### Creating resources
You **should** support creating resources using `PUT` and/or `POST`. 

How to choose:

#### Using PUT
You should use `PUT` when you know the name (and therefore location) of the resource you are creating.

Pros:
- Idempotent.
- The caller can provide semantically meaningful name for the resources.

Cons:
- The caller *has* to provide a name for the resource even if there is no natural semantically meaningful name for the resource.

> If you support `PUT` for creating resources, you may additionally support creating (upserting) resources using `PATCH`.

You **may** support both PUT, PATCH and POST for creating the same resource. 

#### POST

Use POST to create resources when the name (and therefore location) is determined by the service. The POST operation should be against the collection of the target resource:

`http://api.contoso.com/exampleservice/widgets`

Return a `location` header indicating where the created resource is located.

1. Creating resources
0. Deleting resources
0. Updating resources (verb, content type etc.)
1. Paginated results
2. Long running operations
3. Throttling and retries
4. Expanding responses
5. Deleting resources
6. Queries and filters
7. RBAC and permissions
8. Caching
9. Versioning
10. Designing for reach (precisions etc.)
0. Arrays vs. objects
0. Media types and versioning

### Entity tags (ETags)

#### Creation of
ETags for Azure services should be strong validators.

The `ETag` value should be a hash of the current representation of the resource it represents.

The `ETag` value *may* be a hash of the `Last-Modified` value, but this has two major problems; 1) the resolution of `Last-Modified` is only down to the second, meaning that two modifications within one second will result in the same `ETag` value unless additional salt is added and 2) it still requires that the `Last-Modified` value doesn't change unless a request actually changes the representation (i.e. you cannot simply update it on every write request)

Using a new random value for ETags would solve the problem with the second resolution limitation of `Last-Modified`, but in order to 

#### ETags for composite resources

#### ETags for Collections

While it is theoretically possible to have a `ETag` for a collection of resources, it is generally not as useful as an `ETag` for an individual resource. 

## A canonical service implementation

- Repository
- Pull request

Paths:

`/service/repositories`

`/service/repositories/{name}`

`/service/repositories/{name}/pullrequests`

`/service/repositories/{name}/pullrequests/{id}`

`/service/repositories/{name}/pullrequests/{id}/reviews`

Creating a repository is done using a PUT operation against the `/service/repositories` collection since the name is semantically meaningful and unique, and thus is something that it is reasonable that a client provides. 

Creating a new pull request is done using a POST operation against the `/service/repositories/{name}/pullrequests` collection, and a unique id is generated by the service. 

Creating a new repository:

### Long running operations

## Long running operation that creates resource

- **DO** follow the [hybrid model](https://github.com/microsoft/api-guidelines/blob/vNext/Guidelines.md#1323-post-hybrid-model) when creating resources.

Specifically:

- **DO** return a 201 - Created status code for initial request (POST, PUT or PATCH). Do include the properties of the newly created resource in the response body.
- **DO** provide a `status` property in the resource that tracks its state.
- **DO NOT** provide a corresponding `Operation` (located by the `Operation-Location` header in the response) resource tracking the creation of the resource unless it is required to track additional state for the resource creation. This is rare.
- **DO** respond with a 200 - OK status code if a client does a `GET` on the newly created resource. In other words, the resource should be visible on its target location immediately after the service has responded with a 201 for the initial request.

TODO: insert call requests/responses here...

## Other long running operations

- **DO** provide an `Operation` resource (and corresponding `Operation-Location` header if one ore more are true:
    - multiple operations can be running in parallell for a given resource/path.
    - the result of the operation is one or more computed values.
- **DO NOT** provide an `Operation` resource if:
    - only one operation can be run at a time. Instead, track the status on the target resource itself. 
        - Example: you have to finish rebooting a virtual machine before you can reboot it again.
# Appendix A - HTTP status codes
|Status code|Meaning|
|200|Success|
|201|Successfully created a new resource|
|204||
