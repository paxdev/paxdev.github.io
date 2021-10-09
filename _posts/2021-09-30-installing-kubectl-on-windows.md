---
  title: Installing kubectl on Windows
  categories:
    - kubernetes
  tags:
    - kubernetes
    - powershell
    - chocolatey
---

**NB**
This post assumes you are using `Powershell`. If not adjust for your shell of choice.

The simplest way to install `kubectl` on Windows is to use [chocolatey](https://chocolatey.org/).

```powershell
choco install kubernetes-cli
```

This installs both `kubectl` and `kubectl-convert` into the `$env:ProgramData\chocolatey\bin` folder.

You will also need a suitable `kubeconfig`.

Change to your home directory

```powershell
cd ~
```

Create a `Kube` folder

```powershell
mkdir .kube
```

For developing on a local cluster you can copy the `kubeconfig` from your Kubernetes control node using `scp`, however in a Production environment you would want to carefully craft a suitable `kubeconfig` with minimal priveleges.

```powershell
scp your-user@your-cluster:~/.kube/config ./.kube
```

In my case I also had to edit this file to replace my control-node's *internal* IP address (that it uses to connect to the worker nodes) to the *external* address it uses to connect to my router.

You can check everything is working by checking the running nodes:

```powershell
kubectl get nodes
```

By default `kubectl edit` will use `Vim` as the editor. For bonus points you can configure an editor of your choice.
This is done by setting the `KUBE_EDITOR` environment variable. 
To use `VSCode` (assuming it is already in the `PATH`):

```powershell
[System.Environment]::SetEnvironmentVariable('KUBE_EDITOR','code -w')
```

The `-w` flag is an abbreviation of `--wait` and tells VS Code to wait for the file to be closed before returning control back to `kubectl`

#### Gotcha

There is a possible gotcha. If you have previously installed `Docker` this also comes with a version of `kubectl` and you may find that the `Docker` folder is before the `chocolatey` folder in your `PATH`. This means that any Command Line will search the `Docker` folder first for `kubectl` and likely try to use the incorrect version leading to errors that look like:

```shell
error: schemaError(x.y.z): invalid object does not have additional properties
```

You can easily check this by using `kubectl version`. You will likely see that the client version is much lower than the server version.

To solve the problem edit your environment variables and ensure that in the `System Path` the `$env:ProgramData\chocolatey\bin` appears before the Docker folder (in my case `$env:ProgramFiles\Docker\Docker\Resources\bin`).

You will now need to close and re-open any shells to get the correct updated `PATH`.
