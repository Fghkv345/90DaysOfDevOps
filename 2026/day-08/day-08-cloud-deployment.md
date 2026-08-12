 <img width="1366" height="768" alt="Screenshot from 2026-08-12 22-20-04" src="https://github.com/user-attachments/assets/85ac2143-274d-48de-a9ba-8f017b3f1db2" />

<img width="1366" height="768" alt="Screenshot from 2026-08-12 22-20-28" src="https://github.com/user-attachments/assets/a5c01dde-6399-48d0-9fc0-f3e1c3a1924e" />

<img width="1366" height="768" alt="Screenshot from 2026-08-12 22-25-54" src="https://github.com/user-attachments/assets/e95dda64-83cb-420a-9de0-b6ca580111c3" />

 

## Commands Used

 
# Check Nginx service status
systemctl status nginx

# Stop Nginx
sudo systemctl stop nginx

# Start Nginx
sudo systemctl start nginx

# Check whether Nginx is listening on HTTP port 80
sudo ss -lntp | grep ':80'

# Test the webpage locally through Nginx
curl http://localhost

# Check the EC2 instance's current public IP
curl -4 ifconfig.me

# Test the webpage using the EC2 public IP
curl http://98.80.14.207

# Check Ubuntu firewall status
sudo ufw status

# Test HTTP connectivity with a timeout
curl --connect-timeout 5 http://98.80.14.207/

# Verbose HTTP connection test
curl -v --connect-timeout 5 http://98.80.14.207/
 

## Challenges Faced

* Initially, `systemctl stop nginx` returned **"Authentication is required"** because the command was executed without `sudo`. Using `sudo systemctl stop nginx` solved the permission issue.
* After starting Nginx, the service was confirmed as `active (running)` and listening on port `80`.
* The webpage initially returned `ERR_TIMED_OUT` in Brave. The AWS Security Group was checked and HTTP port `80` was allowed from `0.0.0.0/0`.
* The EC2 instance was restarted, which changed its public IPv4 address from `3.88.63.78` to `98.80.14.207`. The new public IP had to be used.
* `ufw` was checked and found to be **inactive**, ruling out the Ubuntu firewall as the cause.
* `curl http://localhost` successfully returned the `index.html` content, confirming that Nginx was correctly serving the webpage.
* `curl http://98.80.14.207` also returned the `index.html` content, confirming that the EC2 instance was reachable through its public IP.
* The webpage opened successfully in Chrome but not in Brave, showing that the AWS, Nginx, and webpage configuration were working and the remaining issue was specific to Brave.

## What I Learned

* `systemctl` is used to manage Linux services, while `sudo` is required when performing privileged service-management operations.
* Nginx can serve static files from `/var/www/html`, with `index.html` being the default webpage.
* `sudo ss -lntp | grep ':80'` can be used to verify that Nginx is listening for HTTP connections on port `80`.
* AWS EC2 Security Groups control inbound traffic, so HTTP port `80` must be allowed for a public web server.
* An EC2 instance's public IPv4 address can change after a stop/start unless an Elastic IP is associated with the instance.
* Troubleshooting should be performed layer by layer: verify the application, service, listening port, host firewall, AWS Security Group, public IP, and finally the client/browser.
