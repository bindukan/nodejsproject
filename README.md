# **nodejs deployment**

## Task: **Deploy nodejs application on docker container**

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

