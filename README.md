# konakart-ecommerce
Docker implementation of Java-based ecommerce platform KonaKart with NGINX reverse proxy in AWS

[insert arch diagram]

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
1. Update packages (both VMs)
   ```
   sudo dnf update
   ```

    *dnf is successor to YUM

2. Install Docker (both VMs)
   ```
   sudo dnf install -y docker  # install Docker
   sudo systemctl start docker  # start Docker
   sudo systemctl enable docker  # enable Docker to automatically restart upon reboot
   sudo systemctl status docker  # verify status of Docker is running
   sudo docker run hello-world  # test running hello-world container
   ```
   
3. Set up non-root user for Konakart installation (Konakart VM)
   ```
   sudo adduser [username]
   sudo passwd [username]
   sudo usermod -aG docker konakartuser  # add user to docker group for use without sudo
   sudo usermod -aG wheel konakartuser  # grant sudo privileges to user
   su [username]  # switch to non-root user
   
4. Pull and run Konakart (Konakart VM)
   ```
   docker run -d --name kk9600 -p 8780:8780 -p 8783:8783 konakart/konakart_9600_ce  # pull and run community version of KonaKart from DockerHub
   ```
   This Docker image contains the KonaKart demo store (Community Edition) with a pre-populated PostgreSQL database.
   After running the `docker run` command open a browser and see the storefront at:

   ```
   http://[ec2-instance-public-ip-address]:8780/konakart/
   ```
   Open admin access at:
   ```
   http://[ec2-instance-public-ip-address]/konakartadmin/
   ```
   Login using “admin@konakart.com” as the username and “princess” as the password.

5. Configure NGINX reverse proxy (NGINX VM)
   ```
   docker run -d --name nginx-base -p 80:80 nginx:latest  #  Run nginx docker container based on NGINX base image
   mkdir nginx  # make directory for NGINX config
   docker cp nginx-base:/etc/nginx/conf.d/default.conf ~/nginx/default.conf  # copy the default.conf file so we can modify it to configure NGINX as a reverse proxy
   vim nginx/default.conf  # edit the default.conf file to map NGINX to Konakart
   docker cp ~/nginx/default.conf nginx-base:/etc/nginx/conf.d/  # copy conf.d file into NGINX container
   docker exec nginx-base nginx -t  # Validate file was copied to nginx-base container
   docker exec nginx-base nginx -s reload  # Reload file into nginx-base container so it has the latest updates
   
   
   
   
   ```
   
