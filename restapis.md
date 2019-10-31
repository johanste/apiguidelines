# Azure REST APIs
The Microsoft REST API guidelines provides general guidelines (requirements) for the design of REST APIs across Microsoft. That there are several "styles" of APIs (e.g. OData, REST, RPC), and the general guidelines provides options for each.

This document is intended to provide a normative description and recommendations specifically for Azure REST APIs. It is intentionally more prescriptive than the general guidelines.

## TLDR

- **DO** use query string version parameters unless your family of services has overridden this. 
- **DO** use date-based API versions.
- **DO** append a -preview suffix for API versions that are in preview.
- **DO** use the correct HTTP verbs for your operation.
- **DO** use PUT against the canonical path of a resource in order to create or replace it if the name of the resource is known a priori.
- **DO** use POST against the canonical path to the collection to create an item in the identity (name) of the created resource is not known a priori.
- **DO** use the same model as request body and response content for any given URL.
- **DO NOT** use semantically meaningful nulls in your stored resources. That is, do not distinguish between a null value for a description of a resource and not having a description for the resource.
- **DO** support updates of all mutable properties of resources using PATCH.
- **DO** use arrays only for collections where order is relevant *or* where the items in the collection does not have a name/identifier.
- **DO** use maps for collections of named resources rather than arrays of objects.
- **DO** use JSON merge-patch as the content type for PATCH operations against resources.
- **DO** document which properties can be updated after a resource has been created.
- **DO** support conditional requests (`if-match`, `if-none-match` and `if-not-modified` headers) for your API. See [optimistic concurrency](#restapi_optimistic_concurrency) for more information.
- **DO** support ETags for your resources.
- **DO** use a stable checksum of the resource representation as the `ETag` value. Do *not* use a random value or last modified time (or derivation thereof) as the value.
- **DO** document your top level error codes and under what circumstances they can occur. 
- **DO** document what property changes will cause service interruption/downtime (e.g. compute resource will be rebooted)
- **DO** document if your POST or PATCH operation is idempotent. See also: the [`x-ms-idempotent`](#traits_idempotent) trait. TODO: Add support for new extension
- **DO** model child resources as collections of the parent resource when the relationship is a composition.
- **DO** allow direct references of child resources for composite resources.


## How to design your API

### Describe your resources.
REST APIs are, by definition, resource centric. Resources represent the nouns in your system.

> The line between nouns and verbs can sometimes be blurry - see [actions](#resource_actions) below...

Since users of your service will be interacting with your resources, it is important that they map to the users conceptual model of the service. A good litmus test is that you should be able to use the name of the resources in the blog post that introduces your service to your users.

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

### 1. Use the correct verbs
### 2. Optimistic concurrency using etags/if-match
### 3. Status codes
### 4. Headers

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
