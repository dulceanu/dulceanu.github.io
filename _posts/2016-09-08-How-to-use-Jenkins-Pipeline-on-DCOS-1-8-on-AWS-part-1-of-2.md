---
layout: post
title: How to use Jenkins Pipeline on DC/OS 1.8 on AWS (Part 1 of 2)
author: andrei
comments: true
category: scalability
---

### What is Jenkins Pipeline

Jenkins Pipeline is a plugin which allows developers to programmatically define Jenkins jobs with simple Groovy scripts. Or, [in their own words](https://jenkins.io/doc/pipeline/),

> While standard Jenkins "freestyle" jobs support simple continuous integration by allowing you to define sequential tasks in an application lifecycle, they do not create a record of execution that persists through any planned or unplanned restarts, enable one script to address all the steps in a complex workflow, or confer the other advantages of pipelines.

> In contrast to freestyle jobs, pipelines enable you to define the whole application lifecycle. Pipeline functionality helps Jenkins to  support continuous delivery (CD). The Pipeline plugin was built with requirements for a flexible, extensible, and script-based CD workflow capability in mind.

----

### Installing and Configuring Jenkins Pipeline

* Install DC/OS 1.8 as instructed [here](https://dcos.io/docs/1.8/administration/installing/cloud/aws/).
* On your local machine install CLI (last step in above link).
* Install a user-specific Marathon instance. This will serve as the deployment platform for any user-created applications that are deployed by Jenkins.

```bash
$ dcos package install marathon
```

* Copy the pem file used in creating the template to your .ssh directory, change permissions, and add it to your known keys.

```bash
$ cp your.pem ~/.ssh
$ chmod 600 your.pem
$ ssh-add ~/.ssh/your.pem
```

* To SSH (from local machine) to a master node:

```bash
local$ dcos node ssh --master-proxy --leader
```
* To SSH (from local machine) to an agent node:

```bash
local$ dcos node ssh --master-proxy --mesos-id=<mesos-id>
```

* Create a new elastic file system, using the vpc created when setting up DC/OS, and when prompted add the security groups of your master and slave.

![EFS Creation]({{ site.url }}/assets/jenkins-pipeline/efs_creation.png)

* Go to master and create the mount target folder:

```bash
$ dcos node ssh --master-proxy --leader
$ sudo mkdir -p /mnt/jenkins
```

* Manually mount the file system to check if it works:

```bash
master$ sudo mount -t nfs4 -o nfsvers=4.1 eu-west-1.fs-XXXXXXXX.efs.eu-west-1.amazonaws.com:/ /mnt/efs
```

* You can then create a sample file in that directory to see later if it's visible from a node:

```bash
master$ cat <<EOT >> /mnt/efs/test.txt
This is a test.
EOT
```

* Now, since this already worked, all we need is to make the EFS mount persistent across restarts. For that we'll have to edit our cloud config:

```bash
master$ sudo vi /usr/share/oem/cloud-config.yml
```

Add the following lines to your **cloud-config.yml** file, under units:

```bash
  - name: rpc-statd.service
    command: start
    enable: true
  - name: mnt-jenkins.mount
    command: start
    content: |
      [Mount]
      What=eu-west-1b.fs-XXXXXXXX.efs.eu-west-1.amazonaws.com:/
      Where=/mnt/efs
      Type=nfs
```

* Repeat the previous step for each of the slaves.

* Restart master and slaves.

```bash
$ sudo reboot
```

* From one of the slaves try to access the file previously created on the master. If everything is OK you should see the following output:

```bash
slave$ cat /mnt/efs/test.txt
This is a test.
```

* On your local machine create a file called **jenkins-config.json** with the following config:

```
{
    "service": {
        "name": "jenkins"
    },
    "storage": {
        "host-volume": "/mnt/efs"
    }
}
```

* Install Jenkins

```bash
local$ dcos package install jenkins --options=jenkins-config.json
```

* Go to DC/OS and open Jenkins:

![Open Jenkins]({{ site.url }}/assets/jenkins-pipeline/dcos_jenkins.png)

* Go to **Manage Jenkins ->  Manage Plugins -> Available plugins**. Make sure that **Pipeline Plugin** is installed, otherwise install it and restart Jenkins. If everything is OK you should have similar to:

![Open Jenkins]({{ site.url }}/assets/jenkins-pipeline/jenkins_plugins.png)

In the next post we'll see how to configure a Pipeline job and what it takes to build a simple Java project with Maven using Jenkins Pipeline.

Useful links:

[1] [https://dcos.io/docs/1.8/administration/installing/cloud/aws/](https://dcos.io/docs/1.8/administration/installing/cloud/aws/)

[2] [https://docs.mesosphere.com/1.8/administration/sshcluster/](https://docs.mesosphere.com/1.8/administration/sshcluster/)

[3] [https://dcos.io/docs/1.8/usage/tutorials/jenkins/](https://dcos.io/docs/1.8/usage/tutorials/jenkins/)

[4] [https://coreos.com/os/docs/latest/mounting-storage.html](https://coreos.com/os/docs/latest/mounting-storage.html)

[5] [https://dcos.io/docs/1.8/administration/storage/nfs/#part2](https://dcos.io/docs/1.8/administration/storage/nfs/#part2)
