# Day 08 – Cloud Server Setup: Docker, Nginx & Web Deployment

### Task *Today's goal is to **deploy a real web server on the cloud** and learn practical server management.*


## Commands Used

- Step 1 : Went to AWS EC2 services
- Step 2 : Went to Instances , select my instace and connected to it
- Step 3 : Connected to an instance using the browser-based client.
- Step 4 : Install Nginx `sudo apt install nginx`
- Step 5 : Verify Nginx is running or active `systemctl status nginx`
- Step 6 : *Test Web Access :* Open browser and visit you Public IPv4 address `http://44.200.57.134`
- Step 7 : To view nginx logs run `journalctl -u nginx` or `journalctl -u nginx -n 10` to see recent 10 logs only
- Step 8 : Save these logs to a file `journalctl -u nginx > nginx-logs.txt` 
- Step 9 : Download this Log File to Your Local Machine
    - Open git bash / cmd terminal / powershell or any terminal in your local machine
    - Locate to the place where your .pem key is present
    - Run `scp -i Key_Name ubuntu@Your_Public_Ip:~/nginx-logs.txt "Location where you want to download the file"`
    - Your download will be completed
- Step 10 : Happy Learning : )


## Challenges Faced

- Faced issue on testing web access for nginx , as I pasted the public ip the browser put https before the ip which resist it to show the page as it is not secured ip ---> then figured out and remove 's' from 'https' and boom! test runs successfully

- Faced some issue in downloading the file , as I was entering wrong .pem file name , entering private ip after ubuntu@__ then asked to claude and realised that I have to put public ip there ---> problem solved successfully


## What I Learned

- I learned how to run server and test it's web access , like for every particular instance we have to configure it's security 
    - How to do it (On AWS EC2):
        - Go to EC2 → your instance → Security tab
        - Click the Security Group → Edit Inbound Rules
        - Add rule: Type = HTTP, Port = 80, Source = 0.0.0.0/0

- **Note** : Security Group ---> A whitelist of allowed connections. Without opening port 80, the Nginx server runs fine internally — but the internet simply can't reach it. The security group is the gate; we have to unlock it.


- I learned that these servers do not runs on https (secured http) we have to remove 's' from 'https' and then hit along with public ipv4 address for our linux server

- I learned how to download a file from server to local machine that is using scp command 
    `scp -i Key_Name ubuntu@Your_Public_Ip:~/nginx-logs.txt "Location where you want to download the file"`



