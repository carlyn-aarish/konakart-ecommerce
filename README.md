# konakart-ecommerce
Follow the steps below to deploy Java-based ecommerce platform KonaKart with NGINX reverse proxy using Docker in AWS.

# Architecture

![konakart_arch](https://github.com/carlyn-aarish/konakart-ecommerce/konakart_arch.png)

# Resources
https://www.konakart.com/downloads/community_edition/

# Prerequisites
Deploy two t2.micro (free tier) Amazon Linux 2023 VMs. One VM will run Konakart and the other will run NGINX.

Security Group Inbound Rules for Konakart VM:
* Allow traffic on ports 22, 80, 8780 from your local IP address
* Allow traffic on port 8780 from public IP address of NGINX VM

Security Group Inbound Rules for NGINX VM:
* Allow traffic on ports 22, 80 from your local IP address

# Steps

## 1. Install Docker (both VMs)
   ```
   sudo dnf update  # update packages
   sudo dnf install -y docker  # install Docker
   sudo systemctl start docker  # start Docker
   sudo systemctl enable docker  # enable Docker to automatically restart upon reboot
   sudo systemctl status docker  # verify status of Docker is running
   sudo docker run hello-world  # test running hello-world container
   ```
   
## 2. Install Konakart (Konakart VM)
   Set up non-root user for Konakart installation.
   ```
   sudo adduser [username]
   sudo passwd [username]
   sudo usermod -aG docker konakartuser  # add user to docker group for use without sudo
   sudo usermod -aG wheel konakartuser  # grant sudo privileges to user
   su [username]  # switch to non-root user
   ```
   Pull and run Docker container running Konakart. This Docker image contains the KonaKart demo store (Community Edition) with a pre-populated PostgreSQL database.
   ```
   docker run -d --name kk9600 -p 8780:8780 -p 8783:8783 konakart/konakart_9600_ce  # pull and run community version of KonaKart from DockerHub
   ```
   After running the `docker run` command open a browser and see the storefront at:

   ```
   http://[konakart-ec2-instance-public-ip-address]:8780/konakart/
   ```
   Open admin access at:
   ```
   http://[konakart-ec2-instance-public-ip-address]/konakartadmin/
   ```
   Log in using `admin@konakart.com` as the username and “princess” as the password.

## 3. Configure NGINX as a reverse proxy (NGINX VM)
   ```
   docker run -d --name nginx-base -p 80:80 nginx:latest  #  Run nginx docker container based on NGINX base image
   mkdir nginx  # make subdirectory for NGINX
   mkdir nginx/conf  # make subdirectory for NGINX config
   docker cp nginx-base:/etc/nginx/conf.d/default.conf ~/nginx/conf/default.conf  # copy the default.conf file so we can modify it to configure NGINX as a reverse proxy
   vim nginx/conf/default.conf  # edit the default.conf file to map NGINX to Konakart
   ```

   After root location entry in `default.conf`, add a new location entry to pass traffic to Konakart VM.
   ```
      location /konakart {
        proxy_pass http://[konakart-ec2-instance-public-ip-address]:8780/konakart;
    }
   ```
   The above configuration will pass traffic from `http://[nginx-ec2-instance-public-ip-address]/konakart` >> `http://[konakart-ec2-instance-public-ip-address]:8780/konakart`

   Create custom NGINX image based on above configuration.

   ```
   cd nginx  # change to nginx subdirectory
   touch Dockerfile  # create new file called Dockerfile
   ```

   Edit contents of Dockerfile:
   ```
   FROM nginx:latest 
   COPY conf/default.conf /etc/nginx/conf.d/default.conf
   ```

   Kill default NGINX container.
   ```
   docker ps  # find container ID for nginx container
   docker kill [nginx container ID]  # kill container
   ```
   
   Build custom image and run container.
   ```
   docker build -t nginx-konakart .  # build custom image based on Dockerfile and tag it as "nginx-konakart"
   docker run –name nginx -p 80:80 -d nginx-konakart  # Run container based off our custom image
   ```

   To test access, go to `http://[nginx-ec2-instance-public-ip-address]/konakart` and you should see Konakart app.

   ![konakart_screenshot](https://github.com/carlyn-aarish/konakart-ecommerce/konakart_screenshot.png)
