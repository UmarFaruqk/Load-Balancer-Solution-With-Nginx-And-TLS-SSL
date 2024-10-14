## Load-Balancer-Solution-With-Nginx-And-TLS/SSL
As a DevOps Enginneer you must be a versatile  professional and know different alternative solutions for the same problem. That is why, in this project we will configure a Nginx Load Balancer solution.
It is also extremely important to ensure that connections to your Web solutions are secure and information is encrypted in transit - we will also cover connection over secured HTTP (HTTPS protocol), its purpose and what is required to implement it.
When data is moving between a client (browser) and a Web Server over the Internet - it passes through multiple network devices and, if the data is not encrypted, it can be relatively easy intercepted by someone who has access to the intermediate equipment. This kind of information security threat is called Man-In-The-Middle (MIMT) attack.
This threat is real - users that share sensitive information (bank details, social media access credentials, etc.) via non-secured channels, risk their data to be compromised and used by cybercriminals.
SSL and its newer version, TSL - is a security technology that protects connection from MITM attacks by creating an encrypted session between browser and Web server. Here we will refer this family of cryptographic protocols as SSL/TLS - even though SSL was replaced by TLS, the term is still being widely used.
SSL/TLS uses digital certificates to identify and validate a Website. A browser reads the certificate issued by a Certificate Authority (CA) to make sure that the website is registered in the CA so it can be trusted to establish a secured connection.
In this project we will register our website with LetsEnrcypt Certificate Authority, to automate certificate issuance we will use a shell client recommended by LetsEncrypt - cetrbot.
# Configure Nginx As A Load Balancer
1. Create an EC2 VM based on Ubuntu Server 20.04 LTS and name it *Nginx LB* (do not forget to open TCP port 80 for HTTP connections, also open TCP port 443 - this port is used for secured HTTPS connections) ![reference image](/Pictures/pic1.PNG)
2. Update */etc/hosts* file for local DNS with Web Servers' names (e.g. *Web1* and *Web2*) and their local IP addresses ![reference image](/Pictures/pic2.PNG)
3. Install and configure Nginx as a load balancer to point traffic to the resolvable DNS names of the webservers
1. update the instance *sudo apt update* ![reference image](/Pictures/pic3.PNG)
2. then install nginx *sudo apt install nginx* ![reference image](/Pictures/pic4.PNG)
3. Open the default nginx configuration file *sudo vi /etc/nginx/nginx.conf* and insert the following configuration into the http section ![reference image](/Pictures/pic5.PNG)
4. Restart nginx *sudo systemctl restart nginx* and make sure it up and running *sudo systemctl status nginx*       ![reference image](/Pictures/pic6.PNG)
5. if you access you public IP address in your browser you should see this ![reference image](/Pictures/pic8.PNG)

#  Register a new domain name and configure secured connection using SSL/TLS certificates
In order to get a valid SSL certificate - you need to register a new domain name, you can do it using any Domain name registrar - a company that manages reservation of domain names. The most popular ones are: Godaddy.com, Domain.com, Bluehost.com.
1. Register a new domain name with any registrar of your choice in any domain zone 
2. Assign an Elastic IP to your Nginx LB server and associate your domain name with this Elastic IP ![reference image](/Pictures/pic7.PNG) ![reference image](/Pictures/pic9.PNG)


You might have noticed, that every time you restart or stop/start your EC2 instance - you get a new public IP address. When you want to associate your domain name - it is better to have a static IP address that does not change after reboot. Elastic IP is the solution for this problem. learn how to allocate an Elastic IP and associate it with an EC2 server and how to associate your domain name to your Elastic IP on this page <https://medium.com/progress-on-ios-development/connecting-an-ec2-instance-with-a-godaddy-domain-e74ff190c233>

3. Update A record in your registrar to point to Nginx LB using Elastic IP address ![reference image](/Pictures/pic16.PNG)
4. Check that your Web Servers can be reached from your browser using new domain name using HTTP protocol - *http://<your-domain-name.com>* ![reference image](/Pictures/pic10.PNG) 
5. Update your *nginx.conf* with *server_name www.<your-domain-name.com>* instead of server_name www.domain.com ![reference image](/Pictures/pic11.PNG)
6. Install certbot and request for an SSL/TLS certificate but first make sure *snapd* service is running *sudo systemctl status snapd* ![reference image](/Pictures/pic12.PNG)
7. Install cerbot *sudo snap install --classic certbot* ![reference image](/Pictures/pic13.PNG)
8. Request your certificate (just follow the certbot instructions - you will need to choose which domain you want your certificate to be issued for, domain name will be looked up from *nginx.conf* file so make sure you have updated it on step run this first *sudo ln -s /snap/bin/certbot /usr/bin/certbot* then this *sudo certbot --nginx* ![reference image](/Pictures/pic14.PNG)
9. Test secured access to your Web Solution by trying to reach *https://<your-domain-name.com>*

You shall be able to access your website by using HTTPS protocol (that uses TCP port 443) and see a padlock pictogram in your browser's search string. Click on the padlock icon and you can see the details of the certificate issued for your website. ![reference image](/Pictures/pic15.PNG)
10. Set up periodical renewal of your SSL/TLS certificate so let's edit the crontab file with the following command *crontab -e* and paste the following configuration *** */12 * * *   root /usr/bin/certbot renew > /dev/null 2>&1** though you can always change the interval of this cronjob if twice a day is too often by adjusting schedule expression. ![reference image](/Pictures/pic18.PNG)


**Congratulation!**