# Demo repo

This is a repository to demonstrate a bug in [Gradle Docker Plugin](https://bmuschko.github.io/gradle-docker-plugin/). 
The issue is that committing a container using `DockerCommitImage` to create a new image does not seem to tag the newly created image correctly. 

## Steps to reproduce the bug

**(1)** Start by making sure there are no images with the `test/my-app` tag.

```bash
❯ docker image ls test/my-app
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE

```

**(2)** Run the Gradle build using `./gradlew` (which defaults to the `demo` tasks), that performs the following steps:

```
:demo
\--- :commit -> the container `test-cont` to a new image with the tag `test/my-app:1.0.0-SNAPSHOT`
     \--- :start -> the container `test-cont`
          \--- :create -> a container with the name `test-cont`
               \--- :build -> an image with the tag `test/my-app:1.0.0` from the included `Dockerfile`
```

The `demo` task is finalized by the `remove` task which removes the `test-cont`.
_If_ everything were to go well, we should see two images within the `test/my-app` repository, namely `1.0.0` and `1.0.0-SNAPSHOT`.

Here's the output of running `./gradlew`

```
❯ ./gradlew

> Task :build
Building image using context '/Users/raju/work/projects/gradle-docker-plugin-issue'.
Using images 'test/my-app:1.0.0'.
Step 1/2 : FROM alpine:3.16
 ---> e66264b98777
Step 2/2 : RUN apk add --no-cache unzip
 ---> Using cache
 ---> 79e6dcbca7e6
Successfully built 79e6dcbca7e6
Successfully tagged test/my-app:1.0.0
Created image with ID '79e6dcbca7e6'.

> Task :create
Created container with ID 'test-cont'.

> Task :start
Starting container with ID 'test-cont'.

> Task :commit
Committing image for container 'test-cont'.
Created image with ID 'sha256:9133d78f56855d38027ffbf04ea9e81c6bab4e3d44d82011728956690eac9e16'.

> Task :remove
Removing container with ID 'test-cont'.

BUILD SUCCESSFUL in 4s
5 actionable tasks: 5 executed
```

**(3)** List the images again:

```bash
❯  docker image ls test/my-app
REPOSITORY    TAG       IMAGE ID       CREATED         SIZE
test/my-app   1.0.0     79e6dcbca7e6   3 minutes ago   5.86MB
```

## Observations

- **Note** that while we have `test/my-app:1.0.0` successfully built (from the `Dockerfile`) there is no `test/my-app-SNAPSHOT`.
- _However_, the console output does list `sha256:9133d78f56855d38027ffbf04ea9e81c6bab4e3d44d82011728956690eac9e16`, which does exist—it's just an anonymous image

```
IMAGE ID       REPOSITORY     TAG         SIZE
9133d78f5685   <none>         <none>      5.86MB
```

## Issue details

You can find the details of the issue I filed against [`gradle-docker-plugin`](https://github.com/bmuschko/gradle-docker-plugin/) [here](https://github.com/bmuschko/gradle-docker-plugin/issues/1098)
