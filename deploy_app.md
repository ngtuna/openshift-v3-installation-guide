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

You can tell `new-app` to use a subdirectory of your source code repository by specifying a `--context-dir` flag. Also, when specifying a remote URL, you can specify a Git reference to use by appending `#[reference]` to the end of the URL.

*Example 1: To Create an Application Using the Git Repository at the Current Directory*
```
$ oc new-app .
```
*Example 2. To Create an Application Using a Remote Git Repository and a Context Subdirectory:*
```
$ oc new-app https://github.com/openshift/sti-ruby.git \
    --context-dir=2.0/test/puma-test-app
```
*Example 3. To Create an Application Using a Remote Git Repository with a Specific Branch Reference:*
```
$ oc new-app https://github.com/openshift/ruby-hello-world.git#beta4
```

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
#### Build strategy detection
If `new-app` finds a `Dockerfile` in the repository, it generates a `**Docker**` build strategy. Otherwise, it generates a `**Source**` strategy. To use a specific strategy, set the `--strategy` flag to either `source` or `docker`.
*Example 4. To Force new-app to Use the Docker Strategy for a Local Source Repository:*
```
$ oc new-app /home/user/code/myapp --strategy=docker
```
#### Language detection
If creating a `**Source**` build, `new-app` attempts to determine which language builder to use based on the presence of certain files in the root of the repository:
*Table 1. Languages Detected by `new-app`*
| Language | Files                        |
|----------|------------------------------|
| ruby     | Rakefile, Gemfile, config.ru |
| jee      | pom.xml                      |
| nodejs   | app.json, package.json       |
| php      | index.php, composer.json     |
| python   | requirements.txt, config.py  |
| perl     | index.pl, cpanfile           |
After a language is detected, `new-app` searches the OpenShift server for `image stream` tags that have a supports annotation matching the detected language, or an image stream that matches the name of the detected language. If a match is not found, `new-app` searches the `Docker Hub registry` for an image that matches the detected language based on name.
#### Specifying an image
The `new-app` command generates the necessary artifacts to deploy an existing image as an application. Images can come from *image streams in the OpenShift server*, images in a *specific registry* or *Docker Hub*, or images in the *local Docker server*.
You can explicitly tell `new-app` whether the image is a Docker image (using the `--docker-image` argument) or an image stream (using the `-i|--image` argument).

*Example 7. To Create an Application from the DockerHub MySQL Image:*
```
$ oc new-app mysql
```
*Example 8. To Create an Application from a Local Registry:*
```
$ oc new-app myregistry:5000/example/myimage
```
To create an application from an existing `image stream`, specify the namespace (optional), name, and tag (optional) for the image stream.
*Example 9. To Create an Application from an Existing Image Stream with a Specific Tag:*
```
$ oc new-app my-stream:v1
```
#### Specifying a Template
The new-app command can instantiate a template from a previously stored template or from a template file
*Example 10. To Create an Application from a Previously Stored Template:*
```
$ oc create -f examples/sample-app/application-template-stibuild.json
$ oc new-app ruby-helloworld-sample
```
To use a template in the file system directly, without first storing it in OpenShift, use the `-f|--file` argument or simply specify the file name as the argument to `new-app`.
*Example 11. To Create an Application from a Template in a File:*
```
$ oc new-app -f examples/sample-app/application-template-stibuild.json
```
TEMPLATE PARAMETERS
Use the `-p|--param` argument to set parameter values defined by the template
*Example 12. To Specify Template Parameters with a Template:*
```
$ oc new-app ruby-helloworld-sample \
    -p ADMIN_USERNAME=admin,ADMIN_PASSWORD=mypassword
```
#### Specifying Environment Variables
You can use the `-e|--env` argument to specify environment to be passed to the application container at run time
*Example 13. To Set Environment Variables When Creating an Application for a Database Image:*
```
$ oc new-app openshift/postgresql-92-centos7 \
    -e POSTGRESQL_USER=user \
    -e POSTGRESQL_DATABASE=db \
    -e POSTGRESQL_PASSWORD=password
```
#### Advanced: Multiple Components
#### Advanced: Grouping
The `new-app` command allows deploying multiple images together in a single pod by using the `**+**` separator. The `--group` command line argument can also be used to specify which images should be grouped together.
*Example 19. To Deploy Two Images in a Single Pod:*
```
$ oc new-app nginx+mysql
```
*Example 20. To Deploy an Image Built from Source and an External Image Together:*
```
$ oc new-app \
    ruby~https://github.com/openshift/ruby-hello-world \
    mysql \
    --group=ruby+mysql
```
