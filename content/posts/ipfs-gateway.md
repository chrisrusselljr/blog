---
title: "Tutorial: Hosting a private IPFS Gateway for your NFT Project"
date: 2022-09-10
description: Kubo (ifps-go), Amazon EC2, namecheap, nginx, and SSL. 
author: "Chris Russell"
linktitle: Private IPFS Gateway Tutorial for NFT Metadata
thumbnail: "/img/network.jpg"
draft: false
slug: nft-ipfs-gateway
tags:
 - Tutorial
 - IPFS
 - NFT
---


## Introduction

One of the last steps of an NFT project is genrating and hosting the images and metadata of each token. 

Most commonly, NFT assets are stored in IPFS (Inter Planetary File System). [IPFS](docs.ipfs.io) (InterPlanetary File System) is an open source, peer-to-peer network for storing and sharing data in a distributed file system. IPFS uses content-addressing to uniquely identify each file in a global namespace connecting all computing devices.


An IPFS Gateway resolves access to any requested IPFS  content identifier, allowing browsers and applications access to IPFS content. A common situation is that you want to give public access to your data stored on your privately operated IPFS nodes. 

Using a distributes IPFS allows for your project's data easy to share, harder to hack or censor, and faster to retrieve. Plus, it makes data immutable, which means that once the data is pinned, it can’t be changed.





While there are a number of paid IPFS-hosting services available, hosting youir own gateway will allow your NFT’s metadata to be shared at faster and at a much lower cost (since no servers are involved).

Other benefits to hosting your own private gateway include:

## Pros
- Full control over your data and custome experience.
- Lower cost than using a paid service for our bandwidth needs
- Flexibility to adjust storage to suit performance needs (CPU, RAM, memory)
- Complete control over file storage and accessibility
- Custom subdomains (ipfs.[your-project-here].com)
- Faster uploads/modification of collection data (some servies are heavily rate-limited)
- Private pinning of NFT files, only allowing your project(s) to be shared through the gateway.

## Cons
- Single point of failure unless your data is pinned elsewhere and/or a swarm is set up
- Somewhat complex set-up.

## Into to Pinning
Pinning is the way of telling IPFS to always keep your asset stored somewhere. You'll also be uploading and pinning your files to this gateway for ease of access. 

Once your metadata is uploaded and pinned, it will be easily acessed through your gateway. Click on the link below as an example 👇

`https://ipfs.tetrateras.io/ipfs/QmUCrsPtFsMwpzqHnR75zP3qQwwXM3TmJVgBYtNQtwRfBd/995.json`


## Why SSL
SSL gives our users privacy through the use of digitial encrpyiton as well as trust that they're being served NFT assets directly from our private gateway. So the traffic you're getting from the gateway is 100% authentic.

## Why NGINX
Nginx is an open source webserver that can also be configured for more advance applications such as load balancer, HTTP cache, and reverse proxy. We will use nginx as a basic reverse proxy.

A reverse proxy is a server that sits behind the firewall and will direct requests to the IPFS gateway. A reverse proxy is generally used to increase security and prevent unnecessary traffic being handled by the application. We will use it for basic routing of file requests to the IPFS gateway. IPFS also provides an API port which gives full read/write access to configure the node. Exposing full access to this API is dangerous. We will completely block access however Nginx could be used to provide authenticated or limited access to this api. Additional details on setting up nginx can be found [here](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-16-04).


## Tutorial

### Instance Settings
- Ubuntu 18.04
- t2.small 
- All SSH traffic on port 22
- All HTTP/HTTPS traffic on ports 80/443 respectively (0.0.0.0/0 & ::/0)
- All TCP traffic on port 8080 (gateway) 0.0.0.0/0 & ::/0)
- All TCP traffic on ports 4001 - 4003 (ipfs connection ports) 0.0.0.0/0 & ::/0)
- Storage = files to be stored * 2.2 ex.) 80 gb of assets  = 176 gb block volume aside
- Used EPS GP3 storage (cheapest option)
- Allocate the EC2 instance to an [Elastic IP](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html).

### Domain Name
 - Purchased from namecheap.com (insert affilaie)
 - Set up a [subdomain](https://www.namecheap.com/support/knowledgebase/article.aspx/9776/2237/how-to-create-a-subdomain-for-my-domain/), using the Elastic IP allocated above. 
 - Set up SSL following [this guide](https://www.namecheap.com/support/knowledgebase/article.aspx/9419/33/installing-an-ssl-certificate-on-nginx/).


## Connect to server and install IPFS

1. ssh into VM  
`ssh -i "cryptids-v6-node-west.pem" ubuntu@ec2-3-101-123-139.us-west-1.compute.amazonaws.com`

2. Install kubo (go-ipfs)  
`wget https://dist.ipfs.tech/kubo/v0.14.0/kubo_v0.14.0_linux-amd64.tar.gz`

3. Uncompress  
`tar xvfz kubo_v0.14.0_linux-amd64.tar.gz`

4. Move binary to $PATH  
`sudo mv kubo/ipfs /usr/local/bin`

5. Cleanup  
`rm kubo_v0.14.0_linux-amd64.tar.gz`  
`rm -rf kubo`

6. Check IPFS version  
`ipfs version`  should display `ipfs version 0.14.0`

7. Add IPFS path to user profile  
`sudo vim ~/.profile`  
Add `export IPFS_PATH=/data/ipfs` to bottom   
`source ~/.profile`  
`sudo mkdir -p $IPFS_PATH`  

8. allow current user access to ipfs data   
`sudo chown ubuntu:ubuntu $IPFS_PATH`

9. Initialize ipfs with server configureation:  
`ipfs init -p server`

10. Update gateway's maximum storage. I set this to 90% of my VMs memory.  
`sudo nano data/ipfs/config`    
Update the fields in the config  
`"StorageMax": "XXGB"`  
Can also type:  
`ipfs config Datastore.StorageMax XXGB`  

11. Create systemctl service:  
`sudo nano /lib/systemd/system/ipfs.service`  
Add contents 
```
[Unit]
Description=ipfs daemon
[Service]
ExecStart=/usr/local/bin/ipfs daemon --enable-gc
Restart=always
User=ubuntu
Group=ubuntu
Environment="IPFS_PATH=/data/ipfs"
[Install]
WantedBy=multi-user.target
```
Restart systemctl daemon so it finds the new service:  
`sudo systemctl daemon-reload`  
Tell systemctl that ipfs should be started on startup:  
`sudo systemctl enable ipfs`  
Start ipfs:    
`sudo systemctl start ipfs`  
Check status:  
`sudo systemctl status ipfs`  
Should output:  

```sh
● ipfs.service - ipfs daemon
     Loaded: loaded (/lib/systemd/system/ipfs.service; enabled; vendor preset: >
     Active: active (running) since Sat 2022-08-27 17:08:52 UTC; 4s ago
   Main PID: 1537 (ipfs)
      Tasks: 7 (limit: 2354)
     Memory: 47.0M
        CPU: 595ms
     CGroup: /system.slice/ipfs.service
             └─1537 /usr/local/bin/ipfs daemon --enable-gc

Aug 27 17:08:53 ip-172-31-20-217 ipfs[1537]: Swarm announcing /ip4/127.0.0.1/tc>
Aug 27 17:08:53 ip-172-31-20-217 ipfs[1537]: Swarm announcing /ip4/127.0.0.1/ud>
Aug 27 17:08:53 ip-172-31-20-217 ipfs[1537]: Swarm announcing /ip4/3.101.123.13>
Aug 27 17:08:53 ip-172-31-20-217 ipfs[1537]: Swarm announcing /ip4/3.101.123.13>
Aug 27 17:08:53 ip-172-31-20-217 ipfs[1537]: Swarm announcing /ip6/::1/tcp/4001
Aug 27 17:08:53 ip-172-31-20-217 ipfs[1537]: Swarm announcing /ip6/::1/udp/4001>
Aug 27 17:08:53 ip-172-31-20-217 ipfs[1537]: API server listening on /ip4/127.0>
Aug 27 17:08:53 ip-172-31-20-217 ipfs[1537]: WebUI: http://127.0.0.1:5001/webui
Aug 27 17:08:53 ip-172-31-20-217 ipfs[1537]: Gateway (readonly) server listenin>
Aug 27 17:08:53 ip-172-31-20-217 ipfs[1537]: Daemon is ready
```

## Checkpoint
Load `$PUBLIC_DNS:8080/ipfs/QmS4ustL54uo8FzR9455qaxZwuMiUhyvMcX9Ba8nUH4uVv`in your browser and you should see the docs directory

in our case: `ec2-3-101-123-139.us-west-1.compute.amazonaws.com:8080/ipfs/QmS4ustL54uo8FzR9455qaxZwuMiUhyvMcX9Ba8nUH4uVv`

## Install nginx
`sudo apt update`  
`sudo apt-get dist-upgrade`  
`sudo apt install nginx`  
`systemctl status nginx`  

Close and reboot instance

Ensure packages are up to date:   
`sudo apt update`   
should show   
`All packages are up to date.`  

Install nginx  
`sudo apt install nginx`

Put certs in /etc/ssl. If you've bought your domain with namecheap, you'll need generate a [CSR code and key file](https://www.namecheap.com/support/knowledgebase/article.aspx/9446/14/generating-csr-on-apache-opensslmodsslnginx-heroku/), [activate your SSL certificate](https://www.namecheap.com/support/knowledgebase/article.aspx/794/67/how-do-i-activate-an-ssl-certificate/), [install your SSL certificates on Nginx](https://www.namecheap.com/support/knowledgebase/article.aspx/9419/33/installing-an-ssl-certificate-on-nginx/). This was a challenging step, but if you read through the three links above, you shouldn't run into any problems. 

## Test your gateway
If everything is configured correctly your IPFS node should be running and your node setup as a full public gateway. A user could request any available IPFS file.  
 
Replace with your server’s address. This request will go and request the file from the peer network and return a text file containing : “hello world!”  

`https://<your ip address>/ipfs/QmTp2hEo8eXRp6wg7jXv1BLCMh5a4F3B7buAUZNZUu772j`

## Restrict File access


1. Configure Gateway to only fetch files that are local to your node.  
`ipfs config --json Gateway.NoFetch true`  

2. restart IPFS  
`sudo systemctl restart ipfs`  

3. delete previously fetched files from cache by running garbage collection  
`ipfs repo gc`  

4. Request remote file again. You should get error message “merkledag: not found”  
`http://<your ip address>/ipfs/QmTp2hEo8eXRp6wg7jXv1BLCMh5a4F3B7buAUZNZUu772j`  

5. Create and pin local file to your node. The resulting hash will be displayed    
`echo "hello friend" | ipfs add`  
6. read the contents of the created file    
`ipfs cat Qmbi8UxZdPLsnDSuQLDmqoo7kuTgTkVhHMHuQcz9X77Jdq`    
7. Request the local file using gateway. You should have access to retrieve this file.  
`http://<your ip address>/ipfs/Qmbi8UxZdPLsnDSuQLDmqoo7kuTgTkVhHMHuQcz9X77Jdq`  

## Configure nginx
```sh
erver {
  if ($host = ipfs.tetrateras.io) {
    return 301 https://$host$request_uri;
  }

  listen 80;
  listen [::]:80;

  server_name ipfs.tetrateras.io;
	root /var/www/html;
	index index.html;
	return 404;
}
server {
  server_name ipfs.tetrateras.io;

  listen [::]:443 ssl ipv6only=on;
  listen 443 ssl;

  ssl_certificate /etc/ssl/ipfs_tetrateras_io_chain.crt;
  ssl_certificate_key /etc/ssl/tetrateras.key;

  location / {
    proxy_pass http://127.0.0.1:8080;
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
  }

}
server {
  server_name ipfs.tetrateras.io;

  listen [::]:4003 ssl ipv6only=on;
  listen 4003 ssl;

  ssl_certificate /etc/ssl/ipfs_tetrateras_io_chain.crt;
  ssl_certificate_key /etc/ssl/tetrateras.key;

  location / {
    proxy_pass http://127.0.0.1:4002;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }
}
```

## Send files to VM
Check out filezilla for an easy to use interface for sending files to your VM. 

## Pin files to your ipfs node
`ipfs add --recursive --progress ./gif  `


## Extra Resources

- [A (loosely written) Guide to Hosting an IPFS Node on AWS](https://talk.fission.codes/t/a-loosely-written-guide-to-hosting-an-ipfs-node-on-aws/234)
- [Setting Up IPFS Server With Nginx And Gateway](https://www.geekdecoder.com/setting-up-ipfs-server-with-nginx-and-gateway/)
- [A mostly complete guide to hosting a public IPFS gateway](https://www.reddit.com/r/ipfs/comments/thpgt1/a_mostly_complete_guide_to_hosting_a_public_ipfs/)
- [Tutorial: Setting up an IPFS peer, part II](https://medium.com/textileio/tutorial-setting-up-an-ipfs-peer-part-ii-67a99cd2c5)
- [IPFS: The (Very Slow) Distributed Permanent Web](https://vinta.ws/code/ipfs-the-very-slow-distributed-permanent-web.html)
- [IPFS Server With Reverse Proxy & Restricted Public Gateway](https://friendsoflittleyus.nl/ipfs-server-with-reverse-proxy-restricted-public-gateway/)
- [What Is IPFS and How Does it Store NFT Metadata?](https://blog.thirdweb.com/guides/securing-pinning-your-nft-with-ipfs/)
- [Generating CSR on Apache + OpenSSL/ModSSL/Nginx + Heroku](https://www.namecheap.com/support/knowledgebase/article.aspx/9446/14/generating-csr-on-apache-opensslmodsslnginx-heroku/)
- [How to complete HTTP-based validation ('Upload a validation file' option)](https://www.namecheap.com/support/knowledgebase/article.aspx/10025/68/how-to-complete-httpbased-validation-upload-a-validation-file-option/)
- [Installing an SSL certificate on Nginx](https://www.namecheap.com/support/knowledgebase/article.aspx/9419/33/installing-an-ssl-certificate-on-nginx/)
