# Network Forensic Analysis Report

## Time Thieves 
You must inspect your traffic capture to answer the following questions:

1. **What is the domain name of the users' custom site?**

frank-n-ted.com

![time_thieves_1.JPG](time_thieves_1.JPG)

2. **What is the IP address of the Domain Controller (DC) of the AD network?**

10.6.12.157

![time_thieves_2.JPG](time_thieves_2.JPG)

3. **What is the name of the malware downloaded to the 10.6.12.203 machine?**
 
june11.dll

![time_thieves_3.JPG](time_thieves_3.JPG)

4. Upload the file to [VirusTotal.com](https://www.virustotal.com/gui/). What kind of malware is this classified as?

Trojan

![time_thieves_4a.JPG](time_thieves_4a.JPG)

![time_thieves_4b.JPG](time_thieves_4b.JPG)


---

## Vulnerable Windows Machine

1. **Find the following information about the infected Windows machine:**

    - **Host name:** Rotterdam-PC
    - **IP address:** 172.16.4.205
    - **MAC address:** 00:59:07:b0:63:a4
    
    ![vulnerable_windows_machine1.JPG](vulnerable_windows_machine1.JPG)
   
2. **What is the username of the Windows user whose computer is infected?**

   matthijs.devries
   
   ![vulnerable_windows_machine2.JPG](vulnerable_windows_machine2.JPG)
   
3. **What are the IP addresses used in the actual infection traffic?**

   **185.243.115.84**

   When viewing the Conversations between 172.16.4.205 and other IP addresses, 185.243.115.84 had the most number of packets and largest bytes.
   
   ![vulnerable_windows_machine3.JPG](vulnerable_windows_machine3.JPG)
   
4. **As a bonus, retrieve the desktop background of the Windows host.**

   I found the records which went to/from 172.16.4.205 and had http.content_type contain "image" and then searched the packets for the string "background".
   
    ![vulnerable_windows_machine4_found.JPG](vulnerable_windows_machine4_found.JPG)
   
   ![vulnerable_windows_machine4_viewed.JPG](vulnerable_windows_machine4_viewed.JPG)
   
---

## Illegal Downloads

1. **Find the following information about the machine with IP address `10.0.0.201`**:

    - **MAC address:** 00:16:17:18:66:c8

    ![illegal_downloads1_mac_address.JPG](illegal_downloads1_mac_address.JPG)
    
    - **Windows username:** elmer.blanco

    After reading https://www.wireshark.org/docs/dfref/s/smb_netlogon.html, I thought that `smb_netlogon.user_name` should show the user name \([it did not work](illegal_downloads1_win_user_not_working.JPG)\) and that `smb_netlogon.os_version` should show the OS version \([it did not work](illegal_downloads1_win_os_not_working.JPG)\).

   I repeated the technique from *Vulnerable Windows Machine* Q2.
   
    ![illegal_downloads1_win_user.JPG](illegal_downloads1_win_user.JPG)
    
    - **OS version:** Windows NT 10.0

    Found OS version in User Agent information.
    
    ![illegal_downloads1_win_os.JPG](illegal_downloads1_win_os.JPG)
    
  

2. **Which torrent file did the user download?**

    Betty_Boop_Rhythm_on_the_Reservation.avi.torrent

    ![illegal_downloads2.JPG](illegal_downloads2.JPG)
