# Blue Team: Summary of Operations

## Table of Contents
- Network Topology
- Description of Targets
- Monitoring the Targets
- Patterns of Traffic & Behavior
- Suggestions for Going Further

### Network Topology

The following machines were identified on the network:

![](final-project-setup.png)

- Capstone 
  - **Operating System**: Ubuntu 18.04.1 LTS sever1 tty1
  - **Purpose**: Filebeat and Metricbeat are installed and will forward logs to the ELK machine. This VM is in the network solely for the purpose of testing alerts.
  - **IP Address**: 192.168.1.105
- ELK 
  - **Operating System**: Ubuntu 18.04.4 LTS ELK tty1
  - **Purpose**: Holds the Kibana dashboards. The same ELK setup that you created in Project 1. 
  - **IP Address**: 192.168.1.100
- Kali
  - **Operating System**: Kali GNU/Linux Rolling Release 2020.1
  - **Purpose**: A standard Kali Linux machine for use in the penetration test. Credentials are root:toor.
  - **IP Address**: 192.168.1.90
- Target 1
  - **Operating System**: Debian GNU/Linux 8 target1 tty1
  - **Purpose**: Exposes a vulnerable WordPress server.
  - **IP Address**: 192.168.1.110
- Target 2
  - **Operating System**: Debian GNU/Linux 8 target2 tty1
  - **Purpose**: Exposes a vulnerable WordPress server.
  - **IP Address**: 192.168.1.115
- ML-RefVm-684427
  - **Operating System**: Windows 10 Pro Version 1909
  - **Purpose**: Windows Hyper-V Host
  - **IP Address**: 192.168.1.1


### Description of Targets
[nmap scan of local network](nmap_scan.txt)

The target of this attack was: `Target 1` (192.168.1.110).

Target 1 is an Apache web server and has SSH enabled, so ports 80 and 22 are possible ports of entry for attackers. As such, the following alerts have been implemented:

### Monitoring the Targets

Traffic to these services should be carefully monitored. To this end, we have implemented the alerts below:

#### Alert 1: Excessive HTTP Errors

packetbeat indice:

`WHEN count() GROUPED OVER top 5 'http.response.status_code' IS ABOVE 400 FOR THE LAST 5 minutes`

Alert 1 is implemented as follows:
  - **Metric**: total count of the five most common HTTP response statii over the last 5 minutes
  - **Threshold**: 400
  - **Vulnerability Mitigated**: attempts at DDOS and brute force attacks on passwords
  - **Reliability**: medium - the alert cannot distinguish between attackers and a high number of legitimate requests amd ot only takes 1.33 requests per second to trigger this alert.

#### Alert 2: HTTP Request Size Monitor

packetbeat indice

`WHEN sum() of http.request.bytes OVER all documents IS ABOVE 3500 FOR THE LAST 1 minute`

Alert 2 is implemented as follows:
  - **Metric**: total size in bytes of the request (body and headers) of all documents in the last minute
  - **Threshold**: 3500 bytes
  - **Vulnerability Mitigated**: preventing large uploads particularly binary files
  - **Reliability**: low - a typical HTTP request size is between 700-800 bytes ([Source](http://dev.chromium.org/spdy/spdy-whitepaper)) - it only takes 5 typical requests to trigger this alert so it is likely to trigger false positives often.

#### Alert 3: CPU Usage Monitor

metricbeat indice

`WHEN max() OF system.process.cpu.total.pct OVER all documents IS ABOVE 0.5 FOR THE LAST 5 minutes`

The [Elastic Guide on System fields](https://www.elastic.co/guide/en/beats/metricbeat/current/exported-fields-system.html) states:
```
system.process.cpu.total.pct
The percentage of CPU time spent by the process since the last update. Its value is similar to the %CPU value of the process displayed by the top command on Unix systems.

type: scaled_float

format: percent
```

The impression I have is this value is a percentage and not the fraction of CPU used.

Alert 3 is implemented as follows:
  - **Metric**: the maximum CPU usage (in percent) over all documents during the last 5 minutes
  - **Threshold**: 0.5%
  - **Vulnerability Mitigated**: detecting living off the land attacks which use CPU resources eg running brute force crackers while logged into file system
  - **Reliability**: low - 0.5% seems to be very low threshold but if the virtual machine has a powerful CPU, the baseline CPU use may be normally very low. It is easy for an attacker to exfiltrate password files/hashes to crack on another machine. I took a screen capture of running top on Target 1 and it has a very high idle CPU value.
  ![target1_top.JPG](target1_top.JPG)

###Patterns of Traffic & Behavior

The template doesn't say what should go in this section.

The watcher conditions were met:
![result_conditions_met.JPG](result_conditions_met.JPG)

I wrote a bash scriptcalled `hammer_target1.sh` on Target 1 to trigger two of the watchers:
![hammer_target1.JPG](hammer_target1.JPG)

It triggered two watchers:
![excess_http_errors_watcher_workers.JPG](excess_http_errors_watcher_workers.JPG)
![http_request_size_monitor.JPG](http_request_size_monitor.JPG)

### Suggestions for Going Further
- Each alert above pertains to a specific vulnerability/exploit. Recall that alerts only detect malicious behavior, but do not stop it. For each vulnerability/exploit identified by the alerts above, suggest a patch. E.g., implementing a blocklist is an effective tactic against brute-force attacks. It is not necessary to explain _how_ to implement each patch.

The logs and alerts generated during the assessment suggest that this network is susceptible to several active threats, identified by the alerts above. In addition to watching for occurrences of such threats, the network should be hardened against them. The Blue Team suggests that IT implement the fixes below to protect the network:
- Alert 1: Excessive HTTP Errors
  - **Patch**: block IP addresses of clients that are making the requests by configuring the firewall
  - **Why It Works**: it blocks their access
- Alert 2: HTTP Request Size Monitor
  - **Patch**: modify the alert to detect when a single IP address is generating a request that is excessive then block that IP address
  - **Why It Works**: reduces the number of false positives currently generated by the alerts
- Alert 3: CPU Usage Monitor
  - **Patch**: restrict tools for living off the land attacks to the root user and only permit the root user to install software. Restrict sudo access.
  - **Why It Works**: reduces the likelihood of attackers being able to live off the land
  
- Critical Vulnerability 1: CVE-2001-0554 - Buffer overflow in BSD-based telnetd telnet daemon on various operating systems allows remote attackers to execute arbitrary commands via a set of options including AYT (Are You There), which is not properly handled by the telrcv function.
  - **Patch**: 
  Update ssh:
  ```
  sudo apt-get update
  sudo apt-get install ssh
  sudo apt-get install openssh-server
   ```
  - **Why It Works**: It updates ssh. The newer version of ssh does not have this vulnerability.
- Critical Vulnerability 2: CVE-2017-7679 - In Apache httpd 2.2.x before 2.2.33 and 2.4.x before 2.4.26, mod_mime can read one byte past the end of a buffer when sending a malicious Content-Type response header.
  - **Patch**: 
  Update apache to the latest version:
  ```
  sudo apt-get update
  sudo apt-get install apache2
   ```
  - **Why It Works**: It updates Apache. The newer version of Apache  does not have this vulnerability.
- Critical Vulnerability 3: CVE-2021-26691 - In Apache HTTP Server versions 2.4.0 to 2.4.46 a specially crafted SessionHeader sent by an origin server could cause a heap overflow.
  - **Patch**: 
  Update apache to the latest version:
  ```
  sudo apt-get update
  sudo apt-get install apache2
   ```
  - **Why It Works**: It updates Apache. The newer version of Apache does not have this vulnerability.
- General vulnerabilities 1: weak passwords
  - **Patch**: 
     - enforce a strong password policy by configuring the system to automatically enforce it
     - have an administrative  policy and inform staff that passwords are not used to be reused for multiple systems
     - force michael and steven to change their passwords or disable their accounts if they are no longer employed here
  - **Why It Works**: strong passwords are harder to crack by brute force and reusing passwords makes multiple systems vulnerable. michael and steven's passwords have been compromised and may already be being used by attackers
- General vulnerabilities 2: files containing sensitive data were publicly web accessible
  - **Patch**: 
     - change directory permissions so normal users cannot upload to /var/www/html
     - change directory permissions so directory contents cannot be browsed
     - check contents of files (eg write a cron job to grep for sensitive data)
  - **Why It Works**: prevents normal users from uploading, prevents attackers from seeing contents of directory, mitigates existing files that have been compromised
- General vulnerabilities 3: poor file permission settings
  - **Patch**: 
    - change directory and file permissions so normal users cannot access to configuration files and directories
    - change existing password because the current password has been compromised
  - **Why It Works**: non-administrative users should no have access to configuration files (principle of least priviledge)
- General vulnerabilities 4: sudo permissions 
  - **Patch**: 
     -  reevaluate steven???s sudo permissions
     -  remove python sudo access 
  - **Why It Works**: it tightens sudo priviledges and therefore reduces attack surface
