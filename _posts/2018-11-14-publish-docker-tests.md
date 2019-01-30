---
  title: Publish Docker Build Test Results
  header:
    subheading: Publishing Test Results from a dotnetcore Docker Build in Azure Devops
  tags:
    - tdd
    - azure-devops
    - docker
    - dotnetcore
---

Whilst building our dotnetcore apps inside a container gives us all sorts of advantages including repeatable builds and caching, 
it can present a challenge on a build server since we are running in the context of the Docker container, *not* the Build Server.

A particular challenge is to publish the results of our Unit Tests.

To extract the Unit Tests, I have found the following approach to work.

1. Create an intermediate `test` layer in the `Dockerfile`. This will create a local image which we will spin up to extract the Test Results.

```dockerfile
# Run dotnet tests
RUN dotnet test ((Path to test.csproj)) -c Release -r /TestResults --logger "trx;LogFileName=TestResults.trx"
# Create a named layer
FROM build AS test

# Continue normal dotnet publish
...

# This is the final layer for our actual app
FROM base AS final ... ((normal publish and entrypoint commands))
```

2. Now amend the `docker-compose` file to separately create the test container:

```yaml
services:
  ((myapp)):
    image: ((myapp))
    build:
      context: .
      dockerfile: ((path-to-docker-file))
      target: final
  ((myapp)).tests:
    image: ((myapp))-tests
    build:
      context: .
      dockerfile: ((path-to-docker-file))
      target: test
```

This will build 2 images, one has the final published application, and the second contains the build artifacts, 
including the test results. 

Note that this may contain build secrets etc., so **should not be published to a public repository**.
If you wanted to be really clean, you could create an image to just hold the Test Results.

Note also that this does trigger two Docker builds, but the second one should just be a run through of selecting cached layers as 
nothing will have changed since the first build.

3. We now need to add a `Bash Script task` to our Azure Build Pipeline. Set the script type to Inline

```bash
docker create --name ((myapp))-test-results ((myapp))-tests
docker cp ((myapp))-test-results:/TestResults/TestResults.trx $(System.DefaultWorkingDirectory)
docker rm ((myapp))-test-results
```

This simply spins up an image, copies out the test results and then removes the image.

4. Finally we add a `Publish Test Results` task to the Build so that we can get the Test Results in our Build Logs. 
Make sure to use VSTest result format if you are using `dotnet test`

If you have a build with multiple test files then you will need some way to run them all. On a Linux container you could try something like

```dockerfile
RUN for testfile in $(find . -name "*.Tests.csproj"); do dotnet test -r /TestResults --logger "trx" ${testfile}; done
```

Then in the Bash script your `docker cp` would look like 

```bash
docker cp test:/TestResults/. .
```
