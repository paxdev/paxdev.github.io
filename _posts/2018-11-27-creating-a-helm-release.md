---
  title: Creating a Helm Release for Kubernetes
---

To deploy a web application to Kubernetes using Helm, use the following steps (assuming you have already containerised your application.)

Note this will create a deployment (to create the Pod), a Service (to label the Pod and how to access it as a service) and an Ingress
(to tell the outside world how to access your service.)

1. Create an empty folder to work in. From the command line use

    ```shell
    helm create {your-chart-name}
    ```

1. This will create a set of default files for you to use. First of all we'll look at `Chart.yaml`

    ```yaml
    apiVersion: v1
    appVersion: "1.0"
    description: A Helm chart for Kubernetes
    name: {your-chart-name}
    version: 0.1.0
    ```

    * `apiVersion:` is required and is the version of the _Helm_ api.
    * `appVersion:` is the version of _your_ app that is being deployed in this chart and is optional. This does not need to be SemVer format.
    * `description:` is an optional description of the app being deployed.
    * `name:` is required and is obviously the name of the Chart.
    * `version:` is required and is the SemVer 2 version of the Chart. I typically leave this as it is and set the version when creating an install package as part of a build pipeline.

    You can check all the options in the Kubernetes documentation. 
    Exactly what you choose to enter here will depend on who your target audience is. 
    As I use Helm Charts for Build/Release, I tend not to go crazy on adding details here,  but if you are going to make your Charts publicly available, then you should give your consumers as much information as they need,

1. Now let's look at `values.yaml`

    ```yaml
    replicaCount: 1

    image:
      repository: nginx
      tag: stable
      pullPolicy: IfNotPresent

    service:
      type: ClusterIP
      port: 80

    ingress:
      enabled: false
      annotations: {}
      path: /
      hosts:
        - chart-example.local
    ```

    This is the file that will be used to build the Charts for your `Deployment`, `Service` and `Ingress`. 
    To see how these values are used, you can see the individual templates in the `/templates` folder.

    I also tend to add `fullnameOverride:`, otherwise Helm will create labels based on a concatenation of your release and chart name, 
    which is probably not desirable. 
    If you wish to name your deployed objects by release, you can always create an override of the `fullnameOverride:` that you specify 
    as part of the release process.

    Personally, I like to set a `values.yaml` that will work for a standard local development environment in my organisation. 
    That way when someone pulls the code for the first time it should JustWork&trade; on their machine.

    In the next step we'll create some overrides to use when creating releases for other environments.

1. You can test this locally
    ```shell
    # lint it!
    helm lint {your-chart-name}
    # do a dry run and output the generated yaml
    helm install --debug --dry-run {your-chart-name}
    # take out the --debug --dry-run if you have minikube installed locally and you can check it deploys as expected.
    ```

1. Create a `values.release.yaml`. I tend to like to keep my chart along with my source code in a root folder `helm` 
(so my 'values.yaml' is then `{root}\helm\{my-chart-name}\values.yaml`). 
I then create my overrides at `{root}\helm\values.release.yaml`.
You could create a `values` file for each environment, 
but then you have to change lots of values in lots of places and can't easily spin up new environments from your release pipeline.

1. I now create a Build Pipeline. In the build pipeline I build my Docker image(s) and push to a repository that my Release Pipeline 
can access.

1. The build should now create a Helm package. (You may also need to install/init Helm on your agent first). 
On Azure I have something like this:

    ```shell
    helm package --version $(Build.BuildNumber) ./$(Helm.ChartPath)/
    ```

    where Build.BuildNumber _must_ output something that validates as a SemVer 2, and Helm.ChartPath is a pipeline variable.

1. I now publish the Helm package and the `values.release.yaml` as build artifacts to be picked up by my release pipeline.
(Note that the other build artifact(s) here is my Docker image(s) published to my repository.)

    _For bonus points you can notice that you will probably want to be installing/initialising Helm a lot, 
    as well as repeating the package/publish steps. On Azure extract these to a parameterised **Task Group** and 
    reap the benefit of never having to repeat those tedious build tasks again!_

1. We can now start to build Release Pipelines. 
Again these next few steps are an ideal candidate for extraction to a Task Group so that you can reuse them across your projects.

1. If your cluster cannot access the Docker image(s) you published in your Build Pipeline, copy it(them) to a repository that your
cluster can access.

1. Perform token substitution on your `values.release.yaml` with relevant settings for your environment.

1. You may need to authorise to Helm at this point for AWS you can use Heptio Authenticator for this 
(installing this is an additional release step of course).

1. You can now (finally!) install your Helm Chart substituting in the values for the environment you are deploying to

    ```shell
    helm upgrade --install {helm-release-name} \
			 --namespace {kubernetes-namespace} \
			 --recreate-pods \
			 -f {path-to-values-release-yaml}/values.release.yaml \
			 {path-to-helm-chart}/{chart-name}-$(Build.BuildNumber).tgz 
    ```

    Note that Helm suffixes a hyphen and the version number to the package. I specified my version number from the Azure build.
    You should use suitable values or variables for your Release Pipeline for the other parameters.  
