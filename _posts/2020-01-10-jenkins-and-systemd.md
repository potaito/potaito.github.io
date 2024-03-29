---
layout: post
title:  "Autostart jenkins agent with systemd"
date:   2020-01-10 16:00:00 +0100
categories: pfsense technical
---

This article explains how to run the java jenkins agent as a systemd service.

## Obtaining the agent executable

The jar-file for the agent can be obtained from Jenkins after creating a new node. Clicking on the node reveals instructions with a download link for the agent. It also displays a randomly generated secret valid specificly for this node. In our example we downloaded the agent executable to `/home/jenkins/agent.jar` with 

```shell
cd /home/jenkins
wget https://jenkins.ourgloriouswebsite.com/jnlpJars/agent.jar
```

## Create Systemd Service

Create a new systemd unit with `sudo nano /etc/systemd/system/jenkins_agent.service` which will start the jenkins agent:

```
[Unit]
Descriptions=Jenkins Slave Agent

[Service]
ExecStart=/usr/bin/java -jar /home/jenkins/agent.jar -jnlpUrl https://jenkins.ourgloriouswebsite.com:443/computer/agent_name/slave-agent.jnlp -secret SomeAutoGeneratedSecret12345 -workDir "/home/jenkins"
User=root


[Install]
WantedBy=multi-user.target
```

`User=root` will specify under which username the agent will be started. Jenkins will also launch docker containers using this user. `root` works fine, since it will have the same standard UID (`1000`) for the rootuser inside the docker container as well as on the host system. Any other user on the host, such as `jenkins`, will probably not exist in the docker container unless created in the Dockerfile. This would result in the user being unknown inside the docker container. The command `whoami` for example would fail inside the docker instance: 

```
$ whoami
whoami: cannot find name for user ID 1001
```

Note that `1001` is the UID on the host system with which the jenkins agent was started. Apparently jenkins passes the user running the jenkins agent also to the docker container.


For testing if everything works, simply start the service manually:

```shell
systemd start jenkins_agent.service
```

then check the output with

```shell
systemd status jenkins_agent.service
```

That's it. If everything went well, the service should be up and running without errors. Enable the service such that it is automatically started upon boot: 

```shell
systemd enable jenkins_agent.service
``` 