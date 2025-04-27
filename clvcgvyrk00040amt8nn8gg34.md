---
title: "Deploy Node.js applications on a VPS using Coolify with Dockerfile"
datePublished: Tue Apr 23 2024 14:14:17 GMT+0000 (Coordinated Universal Time)
cuid: clvcgvyrk00040amt8nn8gg34
slug: deploy-nodejs-applications-on-a-vps-using-coolify-with-dockerfile
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1713602895834/024d6d38-4077-446c-aa3d-c99eb774eaff.png
tags: docker, nodejs, devops, self-hosted

---

In the previous article, I shared how to set up Coolify and deploy a Node.js application on a VPS using Nixpacks. It is the easiest way to deploy applications using Coolify. We don't need any knowledge of Docker.

Read Here: [Deploy Node.js applications on a VPS using Coolify](https://sreyaj.dev/deploy-nodejs-applications-on-a-vps-using-coolify)

In this article, we are going to see how to deploy a similar Node.js application using Dockerfile. This is a common way to deploy applications. We create a **Dockerfile**, which can build an image and the image can be deployed to any server.

## Deploy using Dockerfile

So, for this example, we are going to take another node application built using:

* [Fast and low overhead web framework, for Node.js | Fastify](https://fastify.dev/)
    
* TypeScript
    
* [Prisma | Simplify working and interacting with databases](https://www.prisma.io/)
    
* PostgreSQL
    

The difference here is that we have to create a Dockerfile for our application.

### Setting up the PostgreSQL database

Under **Project** &gt; **Environment** &gt; **Resources** &gt; + **New** &gt; **Databases**, you'll be able to see **PostgreSQL**. Start by adding a database into your project.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713453370833/8b2378cd-0898-41d0-881e-ea64c7e1c94e.png align="center")

All the necessary information is prefilled, and we are good to go. Clicking on **Start** will spin up a docker container. Wait for the process to complete and once done, the status would say **Running**.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713601254029/645e2ee9-7e0a-4ca7-a5cc-575d0e8de616.png align="center")

### **Deploying The Node Application**

For deploying our node application, we start by adding a new resource to your project by going to **Project** &gt; **Environment** &gt; **Resources** &gt; **\+ New &gt; Public Repository**

```http
https://github.com/adisreyaj/notes-api-fastify
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713600630812/e2e24f2b-7975-47f4-9910-481c813ebf03.png align="center")

Make sure you have selected **Dockerfile** under **Build Pack** option.

Here's the `Dockerfile` of our sample app:

```dockerfile
FROM node:21-bullseye-slim

ENV DATABASE_URL=${DATABASE_URL}
ENV PORT=${PORT}

WORKDIR /

COPY package*.json ./
COPY . .
ADD prisma .

RUN npm ci
RUN npm run build
RUN npx prisma migrate deploy

EXPOSE ${PORT}

CMD ["npm", "start"]
```

#### **Configure environment variables!**

We need to configure the environment variables that are consumed by our application. Navigate the **Environment Variables** page and start adding the following variables.

Copy the **Postgres URL (internal)** from the database configuration page.

```markdown
postgres://postgres:VrKrSbxvicsQR45w0XTt@e34ds48g44o:5432/postgres
```

Add the following variables: `DATABASE_URL`

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713601731119/fb8ce6a2-8395-429f-956f-dcb201f82ed8.png align="center")

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Make sure to mark <code>DATBASE_URL</code> as <strong>Build Variable</strong> as Prisma will run the migrations in the build phase so we need the variable to be accessible then.</div>
</div>

The value provided in the **Ports Exposes** will be used by the application.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713601851624/432218a9-ad96-43df-976b-6e636c751db6.png align="center")

#### **Deploy the application!**

Once we have configured the env variables, we can go ahead a **Deploy** the application. Coolify shows the logs of the process, we can click on **Show Debug Logs** to see detailed logs.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713601166390/c22a9cd4-31c9-43f4-98cb-4dee61534b04.png align="center")

Once the application is deployed successfully, we can view the logs of the application under the **Logs** tab.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713601938566/1f54f758-d248-4635-85d4-86aaec3cb628.png align="center")

Feel free to reach out to me if you are having trouble in deploying using above mentioned steps. Happy to help.

## Troubleshoot

#### Prisma Migrate Failed

If you are getting the following error:

```bash
ERROR: failed to solve: process "/bin/sh -c npx prisma migrate deploy" did not complete successfully.
```

Make sure that you have provided the correct database URL and also **Build Variable** is checked.

#### Health Check Fail

Make sure the **Health Checks** configuration is correct. If you don't want to run health checks, make sure to turn it off.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1713602589333/4cd160b3-61f4-4c95-8dc5-60b141acb5e6.png align="center")

## Connect with me

* [Twitter](https://twitter.com/AdiSreyaj)
    
* [GitHub](https://github.com/adisreyaj)
    
* [LinkedIn](https://www.linkedin.com/in/adithyasreyaj/)
    
* [Portfolio](https://adi.so)
    

[![Buy me a pizza](https://cdn.hashnode.com/res/hashnode/image/upload/v1639498527478/IA3aJ9R0J.png align="left")](https://www.buymeacoffee.com/adisreyaj)