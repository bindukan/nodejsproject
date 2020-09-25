# Task 1: **nodejs deployment**

## **Deploy nodejs application on docker container**

### prerequisites:

   > Jenkins

   > Docker

*Branches:* (Assumption)

   > development

   > production

   > master

*Environments:*

   > development

   > production

### Deployment steps:

Scripts required for deployment pushed to repository along with the code.

	1. Dockerfile

	2. Jenkinsfile

## Dockerfile:

> create image based on the official Node 12 image from the dockerhub
>
> **`FROM arunlicious/node:12`**
>
> Create a directory where our app will be placed
>
> **`RUN mkdir -p /usr/src/app`**
>
> Change directory so that our commands run inside this new directory
>
> **`WORKDIR /usr/src/app`**
>
> Copy dependency definitions
>
> **`COPY package.json /usr/src/app/`**
>
> Install dependecies
>
> **`RUN npm install`**
>
> Get all the code needed to run the app
>
> **`COPY . /usr/src/app/`**
>
> Expose the port the app runs in
>
> **`EXPOSE 1337`**
>
> Serve the app
>
> **`CMD ["node", "nodeapp.js"]`**

#
## Jenkinsfile:

*Stages:*

- Checkout
- Build docker image using Dockerfile
- Run docker container using image generated in stage2. 
          
Variables used in the pipeline:

- app_name = nodejs
- def env = "${params.appENV}" (Choice parameter)
- def BRANCH = "${params.appBRANCH}" (String parameter)


Jenkins global environment variable:
- WORKSPACE
- BUILD_NUMBER	


### Note: 

	- I am creating the pipeline for both and non prod in single job. It is recommended to have separate pipelines for both prod and non prod.
	- I have not triggered web hook as there is no continuous development. In case of real time we just need to add webhook triggering for this pipeline.
	- Credentials of Jenkins in this project configured on my local machine. 
	- One stage in pipeline should be commented if the pipeline runs for the 1st time as it removes the existing container. It will throw an error if container does not exists.

#
# Task 2:
## Description : The same company xyz has provided one of his developers an oracle based application which runs on a web logic server. Somehow the developer while trying to deploy the application is getting file related errors and he somehow concludes that there are way too many files open in the server via some editor or other commands like tails etc. So DevOps needs to identify the issue of too many open files and solve it.


#
## **Findings:**

The ulimit command allows you to control the user resource limits in the system such as process data size, process virtual memory, and process file size, number of process etc.

> **`$ ulimit -a`**


	>	core file size          (blocks, -c) 0
	>	data seg size           (kbytes, -d) unlimited
	>	scheduling priority             (-e) 0
	>	file size               (blocks, -f) unlimited
	>	pending signals                 (-i) 3802
	>	max locked memory       (kbytes, -l) 16384
	>	max memory size         (kbytes, -m) unlimited
	>	open files                      (-n) 1024
	>	pipe size            (512 bytes, -p) 8
	>	POSIX message queues     (bytes, -q) 819200
	>	real-time priority              (-r) 0
	>	stack size              (kbytes, -s) 8192
	>	cpu time               (seconds, -t) unlimited
	>	max user processes              (-u) 3802
	>	virtual memory          (kbytes, -v) unlimited
	>	file locks                      (-x) unlimited


In linux , Socket connections are treated as files. So any socket connection created by any linux process will be consider one open file.
 
weblogic java process create lot of socket connections, it might create the maximum limit of open files which a process can open.
 
To find max open file limit: 

> **`$ ulimit -n`**

ulimit further divided into soft limit and hard limit.
 
###  *Soft Limit:*
Soft limit means process will be allowed to go beyond this limit but it is warning that you are exceeding your resource consumption. And it will meet your hard limit soon.
> **`$ ulimit -S -n`** 
 
 
### *Hard Limit:*
Hard limit is a full stop for a process, Process will not be allowed to create more connection than this count. 
> **`$ ulimit -H -n`**
 
#
## Solution:


In case of weblogic application running on linux server might creating so many connections like 3k - 4k connections. We have to increase the limit.
The maximum number of file descriptors is controlled two different ways:

- a) Per-User Limit:

1. Explicitly set the number of file descriptors using the ulimit command
> $ **`ulimit -n <open count>`**

ex:
> **`$ ulimit -n 4096`** 

### Note:
ulimit -n command show default soft limit, but when we set this using `ulimit -n <open count>` it set hard and soft. So if hard limit is 8096 and set ulimit -n 4096, it will set hard and soft limit to 4096 for that linux session.

2. Set below values in **`/etc/security/limits.conf`**
 
> soft nofile 65536

> hard nofile 65536
 
The new value added in /etc/security/limits.conf can be taken effect only after logout and login or requires OS restart. 
 
pam-limits
If above changes are not working for then add below value in **`/etc/pam.d/common-session`**

> session required pam_limits.so


- b) System-Wide Limit
Set this higher than user-limit set above **`/etc/sysctl.conf`**

> fs.file-max = 2097152

> **`$ sysctl -p`**

This will will increase total number of files that can remain open system-wide.

### Verify New Limits

> **`$ cat /proc/sys/fs/file-max`**

Hard Limit
> **`$ ulimit -Hn`**

Soft Limit
> **`$ ulimit -Sn`**

Check limit for other user
> **`$ su - <user> -c 'ulimit -aHS' -s '/bin/bash'`**

