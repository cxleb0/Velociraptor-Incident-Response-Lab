# Incident Response Walkthrough

This project documents an end-to-end incident response scenario I carried out using **Velociraptor**. The goal was to execute a complete incident response scenario, in response to 
a simulated malicious persistence attempt. 

---

## 🛠 Preparation  
- Set up and configured my lab environment with Velociraptor's Client-Server deployment.
- Hosted Velociraptor on my Ubuntu server.
- Deployed the client on a Kali Linux VM and connected it to the server.
- I captured screenshots of the Velociraptor admin panel and connected client list to show the baseline state before malicious activity.
![admindashboard](https://github.com/user-attachments/assets/50cd2fac-881e-423a-9e49-6e91e9893e24)
![clientsearch](https://github.com/user-attachments/assets/574f0ce8-91d6-446c-a205-a28fefa94c10)




---

## 🔎 Detection  
- To simulate an attack, I used the [MITRE ATT&CK technique **T1546.004**](https://attack.mitre.org/techniques/T1546/004/) – persistence through malicious shell scripts.  
- Created a persistence mechanism by modifying the client's shell configuration file.
- Note: the persistance mechanism purely servers as a proof of concept. No real malware was created.
  <img width="1918" height="982" alt="malicious behavior" src="https://github.com/user-attachments/assets/93aaedf9-d92b-4b50-af9e-b04b79e6a957" />
 
- From there, I ran an interrogation on the client to get some basic system information, then proceeded onto further analysis.
  ![admin dashboard](https://github.com/user-attachments/assets/edc6c3da-0ffb-4139-a508-fb56441ea83c)


---

## 📊 Analysis  
- Since I knew the persistence technique targeted shell configs, I searched for Velociraptor artifacts that would help me find the exact commands that were executed to establish persistence.
- Using Velociraptor, I ran the **Linux.Sys.BashHistory** artifact to collect shell history from the client.
- This revealed the commands used to establish persistence.  
![Capture](https://github.com/user-attachments/assets/525079ce-10fa-49a9-99c5-5eed23ef8a30)
![artifact result](https://github.com/user-attachments/assets/ea372dc3-63cb-44db-af77-99712dd326af)


- Confirmed the attempts to establish persistence through the modification of the '.zshrc' file.


---

## 🛡 Containment  
- The next step I took was to contain the malicous activity.
- I quarantined the compromised host in Velociraptor.
- This will restrict all outbound communication and only allow it to communicate with the server.
  ![quarantine admin panel](https://github.com/user-attachments/assets/db633557-316f-4927-a27b-0f7e77a6d6ac)
  ![quarantined client](https://github.com/user-attachments/assets/f9260e19-6421-4675-8ff8-873e9de47d0b)
- Began investigating the host remotely using Velociraptor's vShell to inspect the modified .zshrc file.  
- Knowing that the .zshrc file was modified, I wanted to see what modifications were made, so I ran the command:
  ```bash
  cat /home/t0rment/.zshrc
  ```
- This revealed the persistence mechanism added to the file.
  ![found malware](https://github.com/user-attachments/assets/3ec05092-2fcd-4777-81a2-5b5a1ca2eccd)




---

## 🧹 Eradication  
- To clean up the persistence mechanism, I removed the malicious entry from `.zshrc` using `sed`:  

```bash
sed -i '/touch \/tmp\/malware/d' /home/t0rment/.zshrc
```


![malware removal commands](https://github.com/user-attachments/assets/50e15e23-78df-4095-a6e5-068b62fdac3c)
- I then deleted leftover files created by the persistence mechanism:  
```bash
rm /tmp/malware
```
<img width="851" height="177" alt="malware removal " src="https://github.com/user-attachments/assets/5fd822f1-90d1-4882-82aa-4cbc7653ee3a" />


- Verified that the malicious entries were successfully removed.
![malware removed](https://github.com/user-attachments/assets/32027c21-3580-43bb-84f0-4c2a99a85c12)

---

## 🔄 Recovery  
- To harden the system and prevent further modifications to the file, I applied the mitigation technique recommended by the 
  MITRE ATT&CK framework, to restrict file and directory permissions.
- Mitigation ID: M1022
- Made the .zshrc file immutable to block unauthorized modifications:
```bash
chattr +i /home/t0rment/.zshrc
```
![permision removal command](https://github.com/user-attachments/assets/376647b8-ac40-4d0d-a568-417d47e162f0)
- Verified on client VM that making further modifications to .zshrc were blocked.
<img width="853" height="628" alt="removed permissions" src="https://github.com/user-attachments/assets/cf49889e-f8e4-4f5e-85dd-8cf8754fdd20" />

- With mitigations applied, the client was safely unquarantined and restored to normal operation.

---

## 📚 Lessons Learned  
- Reinforced the importance of visibility, speed, and repeatability in incident response. 
- Practiced the full incident response lifecycle by simulating a real-world MITRE ATT&CK technique and responding with Velociraptor.
- My next steps in a production environment would include establishing file integrity baselines to detect and prevent unauthorized modifications
  so an event like this doesn't happen again.
