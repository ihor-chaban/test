

## Task #1 (support-engineer-windows-01 AMI)
- Created new EC2 instance using provided AMI, added tag "name/surname".
- Connected to newly created EC2 instance with RDP, checked that the opsworks.io doesn't work indeed.
- Checked how the domain name is resolving:
`tracert opsworks.io` or `dig opsworks.io @localhost` (dig tool should be installed additionally).
- The domain name doesn't resolve properly, it is linked to 35.158.171.231 IP address.
- Go to /Windows/System32/drivers/etc/hosts and comment out corresponding string `#35.158.171.231 opsworks.io` and save changes.
- Clear local DNS cache `ipconfig /flushdns`.
- The website is working now (but not in IE though, I believe due to JS compatibility issues).
- I have not found VPN credentials in "credentials for l2tp.txt" file on a desktop, so I used my personal l2tp VPN, created a new VPN connection.
- In order to keep RDP connection after the VPN is connected, the original IP address should remain the same.
- Go to Network settings -> Ethernet -> Change adapter options -> open Properties of VPN connection -> Networking tab -> TCP/IPv4 properties -> Advanced -> Unmark "Use default gateway on remote network" and save changes.
- Now the original IP address remains the same even when VPN is connected. RDP connection is live.

## Task #2 (support-engineer-linux-01 AMI)
- Created new EC2 instance using provided AMI, added tag "name/surname".
- Connected to newly created EC2 instance with SSH, checked that the website is not working indeed.
- Check the firewall configuration and routing: <br/>
`ubuntu@ip-172-31-1-79:~$ sudo iptables -S`<br/>
`-P INPUT ACCEPT`<br/>
`-P FORWARD ACCEPT`<br/>
`-P OUTPUT ACCEPT`<br/>
`-A INPUT -p tcp -m tcp --dport 80 -j DROP`
- Check the ID (line number) of the last rule and remove it: <br/>
    `ubuntu@ip-172-31-1-79:~$ sudo iptables -L --line-numbers`<br/>
    `ubuntu@ip-172-31-1-79:~$ sudo iptables -D INPUT 1`
- Open 80 port (Inbound) for the attached Security Group "launch-wizard-55" in AWS console:
![AWS Security Group](https://image.prntscr.com/image/6DGDgnBkTd60CEi6gurKow.png)
- Now 80 port is opened and incoming traffic is allowed, we can see that the browser is connecting to the destination, but Apache returns internal server error.
- Checking Apache error logs: <br/>
`ubuntu@ip-172-31-1-79:~$ tail -n 20 /var/log/apache2/error.log` <br/>
`[Tue Sep 03 10:17:54.032219 2019] [core:alert] [pid 20526] [client 127.0.0.1:55970] /var/www/wordpress/.htaccess: Invalid command 'RewriteEngine', perhaps misspelled or defined by a module not included in the server configuration`
- Enable "mod_rewrite" with the following commands: <br/>
`sudo a2enmod rewrite`<br/>
`sudo service apache2 restart`
- Checking Apache error logs again in the same way: <br/>
`[Tue Sep 03 10:29:51.609279 2019] [:error] [pid 29866] [client 52.58.143.11:11193] PHP Parse error:  syntax error, unexpected 'require' (T_REQUIRE) in /var/www/wordpress/index.php on line 17`
- Go to "/var/www/wordpress/index.php" and add a missing semicolon at line 17, save.
- The website is loading now but seems to be corrupted.
- Checking browser debug console. See that some files can't be loaded and trying to load from strange IP address "18.130.6.35" which is not correct.
- Find the DB credentials from "/var/www/wordpress/wp-config.php" file.
- Log in to MySQL: <br/>
`ubuntu@ip-172-31-1-79:~$ mysql -u wordpress -p**********` <br/>
and search all occurrences of "18.130.6.35" in the database: <br/>
`mysql> use wordpress;`<br/>
`mysql> SELECT * FROM wp_options WHERE option_value LIKE '%18.130.6.35%';` <br/>
and replace update such rows with a valid IP address "52.211.95.229".
- Alternate way. Create mysqldump and replace the IP address in it, then import mysqldump back: <br/>
`ubuntu@ip-172-31-1-79:~$ mysqldump -u wordpress -p********** wordpress > wordpress.sql`<br/>
`ubuntu@ip-172-31-1-79:~$ sed -i 's/18.130.6.35/52.211.95.229/g' wordpress.sql`<br/>
`ubuntu@ip-172-31-1-79:~$ mysql -u wordpress -p********** wordpress < wordpress.sql`
- Now looks better, but still, some links are not valid. Change 'siteurl' in 'wp_options' from 'http://52.211.95.229/s' to 'http://52.211.95.229': <br/>
`mysql> UPDATE wp_options SET option_value =  'http://52.211.95.229' WHERE option_name = 'siteurl';`
- Check everything, click some links, /wp-admin, etc. The website looks good now.

## What is your most significant achievement on the last place of work?
It is hard to determine something particular. I think that the constantly-high KPI is a valuable achievement itself since you make customers happier with your help. Also, I created a few scripts which were not a part of my responsibilities. To my mind, every single extra-mile is a significant achievement. When you are doing more than was expected it's always cool.
## What are some challenges you faced and how did you handle them?
Looking-for a good job is always challenging, I believe. Because it should include interesting and challenging tasks, possibilities for personal and professional growth, friendly atmosphere in team/collective, competitive compensation, etc. How do I handle it? No secrets, just try to give it my best shot. If we are talking about things not related to the job, I think it is my DIY vape powered by Arduino. It was created from scratch (hardware and software) on my own. I'm pretty satisfied with it now, but this pet project is still being improved when I have some spare time and a bit of inspiration. The code can be found on my GitHub (not an ad :D ).

## What’s your experience with Linux operating systems?
My experience with Linux basically consists of my last job as a web-hosting Customer/Technical Support representative. Mostly it is about checking logs, configuring some basic firewall-related stuff, troubleshooting some CMS or mail issues, etc. I know how to use GNU core utilities and often use Linux for my personal purposes (I have dual-boot on my laptop Ubuntu+Windows for different kind of tasks).

## What’s the most complex troubleshooting task you’ve faced recently?
I think it was creating, or better to say, forking new crypto coin (my job before the last one). It was a really challenging task, since I've done almost everything, starting from a suitable coin to fork which fulfill customer's requirements, running a new blockchain from scratch, building the corresponding network, adjusting coin properties, building wallet for different OS (win, linux, mac ), deploying and troubleshoot ongoing issues. I was the only guy working on this task with no experience but little knowledge on this field. Must say, the project was finished successfully and I gained valuable experience, seriously.

## Do you have any experience working as a support technician? Describe your responsibilities, challenges and day-to-day work.

Yes, I worked as a Customer/Technical Support representative. My job was related to a web-hosting (shared servers, VPS, dedicated servers).
These position responsibilities include:
- Provide the first line of customer service through helpdesk and chats.
- Process and resolve client's technical issues within the area of expertise (WHM/cPanel interface, DNS configuration, issues with PHP-based CMS, mail client configuration).
- Troubleshoot and investigate technical issues internal shared servers may have.
- Report bugs/issues found in our systems/servers to the corresponding personnel.
- Update department knowledgebase.
- Escalate issues that are out of a scale of responsibility to the corresponding personnel.
## Do you write any scripts to automate repetitive tasks? Provide an example of such a task and how you’ve approached writing this script.
I created a few simple bash scripts in order to make some routine repetitive actions easier and faster. For example, there was a necessity to read ModSecurity logs and find specific IDs and whitelist them. Such log entries were pretty big and contain a lot of unimportant information which makes such action a kind of eye torture. My script takes input variables like server type, number and domain name, connects to the corresponding server and shows only relevant information in the handy form "{date} {time} {id}". This script can be found here https://ihor-chaban.com/modsec.gz.

## Have you read anything related to IT in general (Operations, Development, QA) recently? Maybe some (online) courses?
Nope. I used to read "Computer Networks" by Andrew S. Tanenbaum, but it was a while ago. So I hardly can remember something useful from there now. Also, I have read a lot of knowledgebase articles (and updated some of them) on my last work. But in general, I like to read some news or tutorials regarding IT and I always ready to learn something new.