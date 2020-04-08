---
layout: post
title:  "Accessing private git repositories with jenkins agents"
date:   2020-01-10 16:00:00 +0100
categories: jenkins
---

The following is a quick tutorial on how to access private git repositories via SSH on Jenkins agents in an efficient way.

There are no issues when cloning public git repositories in Jenkins. Private repositories however need some additional setup. The less sustainable solution is to copy the SSH keys to all jenkins agents and force jenkins to checkout sources via SSH instead of HTTPS. This method has a variety of disadvantages:

- Setting up new jenkins slaves involves an extra step for copying the SSH key.
- When the SSH key is changed, it needs to be updated on all agents individually. Again, extra work which will probably increase the 
- It's not straight forward to force Jenkins to use SSH instead of HTTPS. In git submodules for instance, the protocol is defined as part of the remote's URL, which is contained in the file `.gitmodules`. Changing these URLs would be a hassle. 

So is there a better way to use SSH inside Jenkins for cloning private repositories? Companies ought to do it all the time, right? The answer is yes! There is a very clean solution when using the **multi-branch pipelines in Jenkins**. The setup works as follows:

1. First add the SSH keys that the agents will be using to authenticate to the jenkins master's credential storage. 
2. Next open the configuration page of the multi-branch pipeline. Under **Branch Sources -> Github** select the SSH credentials, which will then be used for cloning the top-level repository. 
3. Now for the submodules, we also need to specify that checkout should occur via SSH. Still in **Branch Sources -> Github**, go to **Behaviours**. Add a new behaviour that is called `Checkout over SSH` and select the SSH credentials to be used. This takes care of two problems: For one it will checkout all submodules over SSH regardless of the originally specified protocol without the need to rewrite the URLs in `.gitmodules`. Nice! Second, Jenkins will now automatically copy the SSH credentials to its connected agents when needed. No additional setup on the slaves is requried. 

I consider the method with multi-branch pipelines and ssh keys the most sustainable option. It allows the SSH key to be updated in one central place. And bringing up a new agent is also simple and straight forward.