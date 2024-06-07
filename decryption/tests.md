nmap -A -T4 <your-ec2-ip-address>
nikto -h <your-website-url>
sqlmap -u "http://your-website-url/path?param=value" --dbs
