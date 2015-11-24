# How to deploy an application to OpenShift v3

## Working with Projects
A project allows a community of users to organize and manage their content in isolation from other communities.
Create new project
```
$ oc new-project <project_name> \
    --description="<description>" --display-name="<display_name>"
```
View projects
```
$ oc get projects
```
Change from the current project to a different project 
```
$ oc project <project_name>
```
Delete a project
```
$ oc delete project <project_name>
```

## Create new applications
You can create a new OpenShift application using the web console or by running the `oc new-app` command from the CLI. OpenShift creates a new application by specifying source code, images, or templates. The `new-app` command looks for images on the local Docker installation (if available), in a Docker registry, or an OpenShift image stream.

If you specify source code, `new-app` attempts to construct a build configuration that builds your source into a new application image. It also constructs a deployment configuration that deploys that new image, and a service to provide load balanced access to the deployment that is running your image.

### Specifying Source Code
The `new-app` command allows you to create applications using source code from a local or remote Git repository. If only a source repository is specified, new-app tries to automatically determine the type of build strategy to use (`*Docker*` or `*Source*`), and in the case of `*Source*` type builds, an appropriate language builder image.


#### Build strategy
There are three build strategies available
* Docker build
* Source-to-Image (S2I) build
* Custom build
By default, Docker builds and S2I builds are supported.
##### Docker build
The Docker build strategy invokes the plain `docker build` command, and it therefore expects a repository with a `Dockerfile` and all required artifacts in it to produce a runnable image.
##### Source-to-Image Build
Source-to-Image (S2I) is a tool for building reproducible Docker images. It produces ready-to-run images by injecting application source into a Docker image and assembling a new Docker image. The new image incorporates the base image (the builder) and built source and is ready to use with the `docker run` command. S2I supports incremental builds, which re-use previously downloaded dependencies, previously built artifacts, etc.

