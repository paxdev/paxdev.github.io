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

One of the disadvantages of doing is, however, is that you need to ensure that you provision your Agent correctly and 
have the complication of ensuring it has the right software/connections.

This is where Docker steps in. We can download a pre-configured agent and run it as a Docker container.
Just make sure that this is run on a machine that is always available, otherwise you ~~may~~ will switch off the machine running your 
Agent and then find that your pipeline has no available Agents in the pool!

```bash
docker run -e VSTS_ACCOUNT=<<my-account>> -e VSTS_TOKEN=<<my-token>> -e VSTS_POOL="<<my-agent-pool>>" -v /var/run/docker.sock:/var/run/docker.sock -it microsoft/vsts-agent
```

* `-e` sets the environment variables your agent will need to communicate with your Azure cloud account.
* `-v /var/run/docker.sock:/var/run/docker.sock` exposes the Docker socket so that you can run Docker commands within the container.
* `-it` run the container in interactive mode.

The problem here is that when you close your SSH session, you can end up killing your running container.

What I do is use `screen`.

Typing `screen` at the command prompt starts a new session. I can then use `Ctrl-A, d` to detach from that screen session which will leave the terminal running when I exit my SSH session.

`screen -r` lists running sessions which I can resume using the name listed. (Even after exiting and starting a new SSH session)

e.g.

```bash
screen -r

There are several suitable screens on:
        7575.pts-0.ubuntu-es    (06/12/18 14:32:50)     (Detached)
        4371.pts-0.ubuntu-es    (19/11/18 16:18:03)     (Attached)
Type "screen [-d] -r [pid.]tty.host" to resume one of them.

screen -r 7575.pts-0.ubuntu-es

```

I can then carry on working, or `exit` this terminal session.
