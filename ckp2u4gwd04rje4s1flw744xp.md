---
title: "How to self-host Plausible Analytics on Ubuntu with SSL for Free!"
datePublished: Mon May 24 2021 16:39:29 GMT+0000 (Coordinated Universal Time)
cuid: ckp2u4gwd04rje4s1flw744xp
slug: self-host-plausible-ubuntu-with-ssl-for-free
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1621875728909/7iIDuZ0HP.jpeg
tags: tutorial, analytics, ubuntu, devops

---

Self-host the open-source alternative to Google Analytics, Plausible using Oracle Cloud Free VM instance. Set up Plausible Analytics on an Ubuntu machine with these simple steps. Follow them one by one and then you can get yourself a simple and easy-to-use analytics tool.

## What is Plausible Analytics?

> Simple and privacy-friendly alternative to Google Analytics

This is what their website says. What makes it better you may ask? For starters, it's open-source. Here are some other features that make it stand out from others:
- Lightweight script (<1KB)
- Simple and easy to use
- Privacy-friendly

Read more about it here: https://plausible.io |  [Demo](https://plausible.io/plausible.io) 

## Self Hosting Plausible

Plausible is open-source. According to their website, there is only one version of it and both the managed version and the self-hosted version are completely equal. 

![Self hosted Plausible](https://cdn.hashnode.com/res/hashnode/image/upload/v1621875592193/YzD1B8mmR.jpeg)

If you go with their managed plan, you'll have to pay a premium that starts at around **$6/mo** for **10k** Monthly page views. You don't have to worry about anything like managing the server, the DB, and stuff.

By self-hosting, you are essentially deploying a version of it on your server. Which means you have to manage the server and other related things on your own. But the bright side of it is that you don't have to pay for using the service.

## How to self host Plausible on Ubuntu?
Today we are going to see how we can host Plausible on an Ubuntu machine. But first, we need to get hold of a VM instance. There are a lot of paid options for you to explore. But if you really just want to try it out, you can opt for Oracle Cloud VM Instance which comes in their Free Tier. 

Among other cloud providers like Google, AWS, Azure, etc there are these lesser know cloud providers like IBM and Oracle which also offer a lot of services in their free tier.
Oracle provides 2 Compute virtual machines with 1/8 OCPU and 1 GB memory each in their ** [Free Tier](https://www.oracle.com/in/cloud/free/#always-free) **

### 1. Creating a VM Instance on Oracle Cloud
First, you need to signup for Oracle Cloud. Once you are in, you can navigate to the `Compute` section then `Instances`.
1. Create a new Instance
Clicking on the `Create Instance` button takes you to a page where you need to select some options.

2. From the Image and Shape settings, choose **Canonical Ubuntu** as the image
3. Add the SSH keys, this will be needed for us to access the instance. (If you are not familiar with SSH keys and how to set them up, Just google it! There are a lot of good articles out there).
4. Create the instance!
5. Once the device is started, we need to open up some ports for the instance. Under Instance Details, click on the `Virtual Cloud Network` link.
6. Navigate to `Security Lists` from the left sidebar.
7. Select the `Default Security List` and make sure we have ports 22, 80, 443 open.
If not, just click on **Add Ingress Rules** and enter the following details:
   - STATELESS: `false`
  - SOURCE TYPE: `CIDR`
  - SOURCE CIDR: `0.0.0.0/0`
  - IP PROTOCOL: `TCP`
  - SOURCE PORT RANGE: `<leave blank>`
  - DESTINATION PORT RANGE: `80,443`
  - DESCRIPTION: `HTTP and HTTPS ports`

![Ingress Rules](https://cdn.hashnode.com/res/hashnode/image/upload/v1621872904923/zSdmmgqsk.jpeg)

### 2. Preparing the Ubuntu Instance
Now that we have set up our cloud instance, we have to prepare the dependencies for Plausible. For that, we need to ssh into the system.

Open up your terminal and ssh into the instance, you can find the details in the **Instance Access** section.
```sh
ssh <username>@<your_public_ip> 
```
### 3. Installing NGINX reverse proxy
For accessing the instance from outside and enabling `HTTPS` on it, we will be using `Nginx` as the reverse proxy. So go ahead and install it on the system.
 
```sh
sudo apt update
sudo apt install nginx
```

### 4. Installing docker
Now we have to install `docker` which is going to run Plausible. Just follow the steps in the official docker guide to install it:
https://docs.docker.com/engine/install/ubuntu/

Once docker is installed, we need to also install `docker-compose`. Follow this guide here:
https://docs.docker.com/compose/install/

### 5. Installing Plausible on Ubuntu
Now that we have set up everything we need to install Plausible, let's dive right in! We'll be following the official guide for installing it:
https://plausible.io/docs/self-hosting

1. Clone the repo to our machine:
```sh
git clone https://github.com/plausible/hosting
cd hosting
```
2. Generate a random 64-character secret key and copy it.
```sh
openssl rand -base64 64
```
3. Update the `plausible-conf.env` file with relevant info. Use any editor on the system to edit the file. Secret key will be the key that we generated in the above step.
```
ADMIN_USER_EMAIL=adi@sreyaj.dev
ADMIN_USER_NAME=adithya
ADMIN_USER_PWD=Str0ngP@assword
BASE_URL=https://plausible.sreyaj.dev
SECRET_KEY_BASE=szKAM+7Vm ... ==
```
*Note:* Try installing `nano` for editing files.

4. Once we have set up the env variables, we can just use `docker-compose` to spin up the system
```sh
sudo docker-compose up -d
```

###  6. Setting up the domain
Now we can go ahead and set up the domain. In our case, we are planning to host it as a subdomain: `https://plausible.sreyaj.dev`.

Go to your domain's DNS console and add an `A` record which points to your instance's public IP.

![Setting up DNS](https://cdn.hashnode.com/res/hashnode/image/upload/v1621872867138/Xc7m1xl91.jpeg)

Once that is done, we need to configure Nginx with the domain. To do so, first copy the Nginx conf file from the repo to the `nginx` folder:
```sh
sudo cp reverse-proxy/nginx/plausible /etc/nginx/sites-available
sudo ln -s /etc/nginx/sites-available/plausible /etc/nginx/sites-enabled/plausible
```
Now edit the file and update the `server_name`
```sh
sudo nano /etc/nginx/sites-available/plausible
```
```
server {
	server_name plausible.sreyaj.dev;
	
	listen 80;
	listen [::]:80;

	location / {
		proxy_pass http://127.0.0.1:8000;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}
}
```
### 7. Setting up HTTPS (SSL) using LetsEncrypt

For setting up SSL, we need to install `certbot`. Follow the steps to install it and add an SSL certificate:
```sh
sudo apt install certbot python3-certbot-nginx
```
Once it is done, run this command:
```sh
sudo certbot --nginx
```
Just enter the required info and then you should be good to go.

### 8. Update IP tables
We need to enable port 80 and 443 on the system, you can enable it by running the command below
```sh
sudo iptables -I INPUT -p tcp --dport 80 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT

sudo iptables -I OUTPUT -p tcp --sport 80 -m conntrack --ctstate ESTABLISHED -j ACCEPT

sudo iptables -I INPUT -p tcp --dport 443 -m conntrack --ctstate NEW,ESTABLISHED -j ACCEPT

sudo iptables -I OUTPUT -p tcp --sport 443 -m conntrack --ctstate ESTABLISHED -j ACCEPT
```

### 9. Completing the setup

Now if you visit the URL, you should be able to see the Login screen for Plausible if everything went well.

Before login, you need to verify the admin user in the DB level:
```sh
docker exec hosting_plausible_db_1 psql -U postgres -d plausible_db -c "UPDATE users SET email_verified = true;"
```

Once this is done, you can log in with the admin email and password provided in the env file above.

### 10. Adding a website to Plausible

Once you are logged in to Plausible, you can add a website. Just enter the URL and select the timezone. Next step you will get the script tag to add to your website. Paste it into your website and you are done.

![Plausible script](https://cdn.hashnode.com/res/hashnode/image/upload/v1621872701446/AqL7VnrJ8.jpeg)

We have finally hosted Plausible on a free Ubuntu VM and successfully connected a website!

Were you able to successfully host your Plausible instance or not, do let me know?

## Connect with me

- [Twitter](https://twitter.com/AdiSreyaj)
- [Github](https://github.com/adisreyaj)
- [Linkedin](https://www.linkedin.com/in/adithyasreyaj/)
- [Cartella](https://cartella.sreyaj.dev) - building at the moment

[![Buy me a pizza](https://cdn.hashnode.com/res/hashnode/image/upload/v1618661389599/2B667-okT.png)](https://www.buymeacoffee.com/adisreyaj)

Do add your thoughts in the comments section.
Stay Safe ❤️