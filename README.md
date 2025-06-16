# DDos Security

![immagine](https://github.com/user-attachments/assets/d9f4a6d4-3012-4e78-b3e3-b97ec30352a6)


##Protect a system, website or online service from being overwhelmed by an enormous amount of malicious traffic coming from multiple distributed sources (often botnets). The goal of a DDoS attack is to make the legitimate service inaccessible to real users.
 
##   Network level defense (Layer 3/4 - IP, TCP, UDP)
## New SYN (TCP SYN Flood) connections:
```
iptables -N DDOS_PROTECTION
iptables -A INPUT -p tcp --syn -j DDOS_PROTECTION
iptables -A DDOS_PROTECTION -m conntrack --ctstate NEW -m limit --limit 10/s --limit-burst 30 -j RETURN
iptables -A DDOS_PROTECTION -j DROP
```
##  Drop anomalous or spoofed packages:
```
iptables -A INPUT -p icmp -m limit --limit 1/second -j ACCEPT
iptables -A INPUT -p icmp -j DROP
```

## level of application defence (Layer 7 - HTTP/S)
## objective of this text, stop the attack targeted to web content (HTTP Flood)
```
http {
  limit_req_zone $binary_remote_addr zone=ddos:10m rate=1r/s;

  server {
    location / {
      limit_req zone=ddos burst=5 nodelay;
    }
  }
}

```
## Fail2Ban for automatic bans on suspicious accesses:
```
[nginx-http-auth]
enabled  = true
port     = http,https
filter   = nginx-http-auth
logpath  = /var/log/nginx/error.log
maxretry = 5
bantime  = 3600
action   = iptables[name=HTTP, port=http, protocol=tcp]
```

## Infrastructure defense (level 7+ - CDN, WAF, cloud protection)
## The objective of this phase is to Offload the load on external infrastructures
##  Benefits:
Layer 3 to Layer 7 protection
Anycast routing to mitigate the attack geographically
Challenge JS or CAPTCHA to filter bots
##  Example with Cloudflare:
Enable the "Under Attack Mode" (challenge for each visit).
Activate WAF rules (e.g.: block all POST from external countries).
Use rate limiting rules es: max 10 requests in 10s.

##  Architectural Defence
##  Resilient load balancer:
```
# Example with HAProxy rate limiting
stick-table type ip size 1m expire 10m store http_req_rate(10s)

```
## Response Monitoring and Automation
##  Bash script to block suspicious IPs:
```
for ip in $(awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head -n 10 | awk '{print $2}'); do
  iptables -A INPUT -s $ip -j DROP
done
```
