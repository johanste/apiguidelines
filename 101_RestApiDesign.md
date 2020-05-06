# Designing your REST API - 101

While every service is unique, following the general guidance in this document helps you getting a good starting structure in your APIs.

## Define your resource types

> TODO: Describe importance of resources (and their names)

Do *not* think about requests and responses for resources. Think about the persisted resource.

## Map your resource types to paths

The canonical path format is:

    /{collection}/{itemId}

The collection segment should be the pluralized name of the resource type: 

    GET /tasks/{taskId}
    
should return a `Task` resource. 

Paths **may** be nested:

    /collection}/{itemId}/{childcollection}/{childId}.
    
See [relationships between resources](##Relationships) for more information.

Avoid nesting deeper than two levels of children (child and grandchild).

### Lists of items vs. reding individual items

> TODO: partial resource in listing vs. full resource when getting. Name + shape.

## Relationships between resources

Prefer to create RelationshipResources over nested resources. For example, a `TaskAssignment` resource can model the relationship between a `Task` resource and a `User` resource. 

> Tip: using a relationship resource allows you to associate additional information with the relationship - for example, when was it created, who created it etc.

Example:

prefer:

    /users
    /users/{userId}
    /tasks
    /tasks/{taskId}
    /taskassignments
    /taskassigmentts/{taskAssignmentId}

over:

    /users
    /users/{userId}
    /users/{userId}/tasks
    /users/{userId}/tasks/{taskId}
    /tasks
    /tasks/{taskId}

If you are unable/it is too unnatural to create a relationship resource, you **may** use a nested read-write child resource or child resource collection. 

Only use child resources this if:

- The items in the collection share the lifetime with the parent resource.
    - Deleting the parent will delete all child resources
    - A child resource cannot be created without a corresponding parent.
 
One consequence of only having nested child resources is that they don't afford any mechanism to list child resources across different parent resources. 

You *MAY* use a nested, read-only collection to help navigating the API. 

## Design your models

## Describe your models in swagger

### Differences between request and response models

## Odds and ends

### Headers vs. body parameters

### Versioning - query parameter vs. path segment vs. content types

> Note: Azure services **MUST** use query string parameters unless grandfathered in to using the version in the path.
