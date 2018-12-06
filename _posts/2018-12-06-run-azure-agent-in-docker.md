---
  title: Run an Azure Agent in Docker
  header:
    subheading: Run an Azure Pipeline Agent as a Docker container
---

There are a number of benefits of running a dedicated Agent for your Azure Pipelines.

1. You can use as much computing power as you can spare!
1. You don't use up any build minutes.
1. You don't have to wait for a Hosted Agent to be provisioned.
1. _(And this is the killer!)_ You can take advantage of Docker caching to speed up your builds.

One of the disadvantages of doing is, however, is that you need to maintain your Agent and make sure it is configured correctly.

This is where Docker steps in. We can download a pre-configured Agent and run it as a Docker container.
Just make sure that this is run on a machine that is always available, otherwise ~~if~~ when you switch it off you will find that your Pipeline has no available Agents!

```bash
docker run -e VSTS_ACCOUNT=<<my-account>> -e VSTS_TOKEN=<<my-token>> -e VSTS_POOL="<<my-agent-pool>>" -v /var/run/docker.sock:/var/run/docker.sock -it microsoft/vsts-agent
```

* `-e` sets the environment variables your agent will need to communicate with your Azure cloud account.
* `-v /var/run/docker.sock:/var/run/docker.sock` exposes the Docker socket so that you can run Docker commands within the container.
* `-it` run the container in interactive mode.

The problem here is that when you close your SSH session, you can end up killing your running container.

What I do is use `screen`.

Typing `screen` at the command prompt starts a new terminal session. I can then use `Ctrl-A, d` to detach from that session, which will stay running if I exit my SSH session.

Later on, when I need to pick up the session, `screen -ls` lists running sessions:

```bash
There are screens on:
        59556.pts-0.ubuntu-es   (23/11/18 13:09:35)     (Detached)
        [...]
Type "screen [-d] -r [pid.]tty.host" to resume one of them.
```

I can then resume with the ID of the session.

```bash
screen -r 59556
```

From here I can carry on working, or `exit` the session to kill it.
