# Example service API design document

## Outline of service capabilities

### Models

* Project
* File
* Build
* ProjectReference (contained)

A project *is associated with* one or more files. 
A project *contains* one or more ProjectReferences.
A file has metadata and contents.
The same file can be *associated with multiple projects*. 
A build *is associated with* a project.

=> We have the following canonical URLs:

/projects {GET, POST}
/files {GET, POST}
/builds {GET, POST}
/projects/{projectName} {GET, PUT, PATCH, DELETE}
/files/{fileName} {GET, PUT, HEAD, DELETE}
/build/{buildId} {GET}

### Verbs

* Add, Delete, Update a reference to a project
* Queue a build
* Search projects

## Scenarios

### Bread and butter

#### 1. Create a new project
- The project has multiple references
- The project contains multiple files

#### 2. Find existing project by name

#### 3. List existing projects

### Possible, but not necessarily easy

#### 1. Add multiple references to a project

