# konakart-ecommerce
Docker implementation of Java-based ecommerce platform KonaKart with NGINX reverse proxy in AWS

# Resources
https://www.konakart.com/downloads/community_edition/

# Prerequisites
1. Deploy two t2.micro (free tier) Amazon Linux 2023 VMs

# Steps
1. Update packages
   ```
   sudo dnf update
   ```

    *dnf is successor to YUM

2. Install Docker
   ```
   sudo dnf install -y docker  # install docker
   sudo systemctl start docker  # start docker
   sudo systemctl enable docker  # enable docker to automatically restart upon reboot
   sudo systemctl status docker  # verify status of docker is running
   sudo docker run hello-world  # test running hello-world container
   ```
   
3. Set up non-root user for Konakart installation
   ```
   sudo adduser [username]
   sudo passwd [username]
   sudo usermod -aG docker konakartuser  # add user to docker group for use without sudo
   sudo usermod -aG wheel konakartuser  # grant sudo privileges to user
   su [username]  # switch to non-root user
   
4. Install and run Konakart
   ```
   docker run -d --name kk9600 -p 8780:8780 -p 8783:8783 konakart/konakart_9600_ce  # pull and run community version of KonaKart from DockerHub
   ```
   This docker image contains the KonaKart demo store (Community Edition) with a pre-populated PostgreSQL database.
   After running the docker run command open a browser and see the storefront at:

   ```
   http://[ec2-instance-public-ip-address]:8780/konakart/
   ```

The Admin Application is at:

http://localhost:8780/konakartadmin/

(login using “admin@konakart.com” as the username and “princess” as the password)

2. Configure nginx reverse proxy

Run nginx docker container based on nginx base image
  ```docker run -d --name nginx-base -p 80:80 nginx:latest```
