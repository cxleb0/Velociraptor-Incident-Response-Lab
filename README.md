# Incident Response Walkthrough

This project documents an end-to-end incident response scenario I carried out using **Velociraptor**. The goal was to simulate a malicious persistence attempt, detect it, and walk through the full response cycle.

---

## 🛠 Preparation  
- I set up and configured my lab environment with Velociraptors Client-Server deployment. 
- I have a machine runnning Ubuntu server where Velociraptor is hosted.
- Once set up and configured, I accessed the admin panel and deployed the client which is a virtual machine running Kali Linux.
- I captured screenshots of the Velociraptor admin panel and conneced client list to show the baseline state before any malicious activity.
![admindashboard](https://github.com/user-attachments/assets/50cd2fac-881e-423a-9e49-6e91e9893e24)
![clientsearch](https://github.com/user-attachments/assets/574f0ce8-91d6-446c-a205-a28fefa94c10)




---

## 🔎 Detection  
- To simulate an attack, I used the [MITRE ATT&CK technique **T1546.004**](https://attack.mitre.org/techniques/T1546/004/) – persistence through malicious shell scripts.  

- I executed a persistence mechanism that modified the client’s shell configuration.
  <img width="1918" height="982" alt="malicious behavior" src="https://github.com/user-attachments/assets/93aaedf9-d92b-4b50-af9e-b04b79e6a957" />
 
- From there, I ran an interrogation on the client to get some basic system information, then proceeded onto further analysis.
  ![admin dashboard](https://github.com/user-attachments/assets/edc6c3da-0ffb-4139-a508-fb56441ea83c)


---

## 📊 Analysis  
- Knowing the persistence technique that was being used, I searched for Velociraptor artifacts that would help me find the exact commands that were executed to establish persistance.
- Using Velociraptor, I ran the **Linux.Sys.BashHistory** artifact to pull shell command history from the client. This revealed the commands used to establish persistence.  
![Capture](https://github.com/user-attachments/assets/525079ce-10fa-49a9-99c5-5eed23ef8a30)
![artifact result](https://github.com/user-attachments/assets/ea372dc3-63cb-44db-af77-99712dd326af)


-I was then able to confirm the attempts to establish persistence through the modification of the '.zshrc' file.


---

## 🛡 Containment  
- The next step I took was to contain the malicous activity.
- In Velociraptor, I quarantined the host which will restrict all network activity and only allow it to communicate with the server.
  ![quarantine admin panel](https://github.com/user-attachments/assets/db633557-316f-4927-a27b-0f7e77a6d6ac)
  ![quarantined client](https://github.com/user-attachments/assets/f9260e19-6421-4675-8ff8-873e9de47d0b)
- This allowed me to investigate the host remotely without the host making other network connections.  
- I then inspected the modified configuration file with Velocriaptor's shell functionality which allows me to run
  commands on the host from Velociraptor
- Knowing that the .zshrc file was modified, I wanted to see what modifications were made, so I ran the command:
  ```bash
  cat /home/t0rment/.zshrc
  ```
- This showed me the file and I was able to identify the malicious entry.
  ![found malware](https://github.com/user-attachments/assets/3ec05092-2fcd-4777-81a2-5b5a1ca2eccd)




---

## 🧹 Eradication  
To clean up the persistence mechanism, I removed the malicious entry from `.zshrc` using Velociraptor's shell functionality  

```bash
sed -i '/touch \/tmp\/malware/d' /home/t0rment/.zshrc
```


![malware removal commands](https://github.com/user-attachments/assets/50e15e23-78df-4095-a6e5-068b62fdac3c)
Then I deleted any leftover files created by the malware:  
```bash
rm /tmp/malware
```

---

## 🔄 Recovery  
To harden the system and prevent further modifications to the file, I used Velociraptor to apply the immutable attribute:  

\`\`\`bash
chattr +i /home/t0rment/.zshrc
\`\`\`

With this change in place, the client was safely unquarantined and restored to normal operation.

---

## 📚 Lessons Learned  
This exercise reinforced the value of **visibility, speed, and repeatability** in incident response. Simulating a real-world ATT&CK technique with Velociraptor helped me practice the full lifecycle of response: from detection to containment, eradication, and recovery.  
