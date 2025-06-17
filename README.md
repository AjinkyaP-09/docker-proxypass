# Handle Traffic Using Docker Containers

In this project I have used a Docker container to handle the traffic on a server. To showcase this load balancing project I have used 3 Docker containers and configured one of them to handle the traffic whereas other 2 as the website.
I have deliberately kept the content of website on both of the containers different to understand and see the working of proxy_pass.

---
# Requirements:
1. docker
2. port 80 in Security Group if using instance (I have used WSL2 environment so no need of that)

# Steps to perform the project:

## Step 1:
- Pull nginx image from DockerHub.
- Use command ```docker pull nginx```.
- Check if image is downloaded.
- Run command ```docker images``` to see present images.
```
apame09@AjinkyaPame:~/$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    1e5f3c5b981a   2 months ago   192MB
```

## Step 2:
- Create our containers from this image.
- Create ```container1``` using ```docker run -d --name container1 nginx```
- Create ```container2``` using ```docker run -d --name container2 nginx```
- Create ```proxycontainer``` using ```docker run -d --name proxycontainer -p80:80 nginx```, don't forget port binding.

## Step 3:
- Create a network and add all of them in that one network.
- Create network: ```docker network create forproxy```
- Check network presence: ```docker network ls```
```
NETWORK ID     NAME       DRIVER    SCOPE
71a18021e502   forproxy   bridge    local
```
- Add each of them to the network: ```docker network network_name container_name```.

## Step 4: 
- Enter container 1 and 2 respectively to add custom webpage content.
- Use command ```docker exec -it container1 /bin/bash``` to enter ```container1```.
- Update apt: ```apt update```
- Install nano to easily put the content: ```apt install nano -y```
- Go to ```/usr/share/nginx/html```.
- Open ```index.html``` and enter content from [/html-pages/cont1-index.html](https://github.com/AjinkyaP-09/docker-proxypass/blob/main/html-pages/cont1-index.html).
- Repeat same for container 2 use content from [/html-pages/cont-2-index.html](https://github.com/AjinkyaP-09/docker-proxypass/blob/main/html-pages/cont2-index.html).

## Step 5:
- Enter in ```proxycontainer``` using ```docker exec -it proxycontainer /bin/bash```.
- Update apt and install nano here too for easy usage.
- Go to configuration pages location of nginx ```cd /etc/nginx/```.
- Open ```nginx.conf```: ```nano nginx.conf```
- Add upstream code as below in http block:
```
upstream webserver {
        server container1:80;
        server container2:80;
    }
```
- File will look like [nginx.conf](https://github.com/AjinkyaP-09/docker-proxypass/blob/main/config-files/nginx.conf).
- Go to ```conf.d``` and open ```default.conf```.
- Enter below lines in ```location /```:
```
        proxy_pass http://webserver;
```
- File will look like [default.conf](https://github.com/AjinkyaP-09/docker-proxypass/blob/main/config-files/default.conf).

### Step 6:
-  Go to browser and put public IP if EC2 instance or just localhost. It will show default nginx page.
![image](https://github.com/user-attachments/assets/59fecb8c-3b21-4f09-94dc-494434353bed)
- Now reload nginx in the container.
- Use ```service nginx reload```.
- Go to browser and put public IP if EC2 instance or just localhost to see the pages.
- Keep on refreshing to alternatively see the pages.
![image](https://github.com/user-attachments/assets/6ff8b692-8f49-430d-86cc-eb059fb8ebd9)
![image](https://github.com/user-attachments/assets/61884640-4af0-4909-9413-1613aac05264)
---

# We have used upstream lets see what it is?
## What is the `upstream` block in Nginx?

In Nginx, the `upstream` block defines a group of backend servers that can handle client requests. It’s commonly used in load balancing setups.

In this project:

```
upstream webserver {
    server container1:80;
    server container2:80;
}
```
This tells Nginx that webserver is a name for a group of two servers (our container1 and container2).

When a request comes to the proxy, it uses this block to forward (proxy) the request to one of those servers.

Nginx automatically load-balances between them using a round-robin strategy (default), so refreshing the page switches between container1 and container2.

Then in the server block:
```
location / {
    proxy_pass http://webserver;
}
```
---
## ✅ Summary

This project demonstrates how to use Docker containers and Nginx to load balance traffic across multiple web servers. With simple containerized architecture, we achieve:
- Traffic routing via reverse proxy
- Basic round-robin load balancing
- Lightweight deployment using official Nginx images

