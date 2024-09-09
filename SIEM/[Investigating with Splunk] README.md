# Investigating with Splunk

## Objective
The Investigating with Splunk project used a controlled environment to analyze logs within a Security Information and Management (SIEM) system to search for data that indicated potential cyber attacks. The project helped enhance understanding of indicators of compromise, attack patterns, and scripting.

### Skills Learned
- Advanced understanding of SIEM concepts and practical application
- Proficiency in analyzing and interpreting event logs
- Development of critical thinking and problem-solving skills in cybersecurity

### Tools Used
- Splunk for log ingestion and analysis 
- Cyberchef for decoding encoded Powershell scripts 

## Scenario
SOC Analyst Johny has observed some anomalous behaviours in the logs of a few windows machines. It looks like the adversary has access to some of these machines and successfully created some backdoor. His manager has asked him to pull those logs from suspected hosts and ingest them into Splunk for quick investigation. Our task as SOC Analyst is to examine the logs and identify the anomalies.
- All the required logs are ingested in the index **main**

### Steps
1. Open Splunk as it will have collected event logs which will allow us to monitor, search, and analyze the aggregated data

   ![image](https://github.com/user-attachments/assets/dee6483b-7626-4116-8831-429cb3385af5)
2. On the left panel, select the "Search & Reporting" tab which will open a new window that will allow you to search through the logs

   ![image](https://github.com/user-attachments/assets/b1bef82d-5d56-4c4f-b55e-852adaf0c74e)
3. Input **index=main** in the search field to see all the events. Change the settings beside the Search menu to:
   - Last 24 Hours --> **All time**
   - Smart Mode --> **Verbose Mode** (to see all events and field data)

   ![image](https://github.com/user-attachments/assets/9392953d-44b3-4884-add9-27de2824071f)
   - The results show there are 12,256 events that were collected in the index main
4. One of the infected hosts seems to have created a backdoor user. To check the logs to see the new username, search using the **EventID=4720**. Scroll down to see more information on the event
   - Event ID 4720 is logged every time a new account is created
   
   ![image](https://github.com/user-attachments/assets/af76ba18-0595-498b-9e28-358a3e04ef39)
   ![image](https://github.com/user-attachments/assets/cc316873-ee77-416e-9e25-a3cdeddd0d90)
   - Only 1 event was logged and the new account name is **A1berto**
5. Look to see who the created user was trying to impersonate by going back to the original search **index=main**, selecting the **User** field in the left pane to see the original users

   ![image](https://github.com/user-attachments/assets/a2d0cb71-f44d-4392-83df-eeacf5b4525b)
   - The user that was impersonated appears to be **Alberto**
6. Check how many times the backdoor user attempted to log in using the **EventID=4624 OR EventID=4625**
   - Event ID 4624 is logged every time an account successfully logs in
   - Event ID 4625 is logged every time an account fails to log in
   
   ![image](https://github.com/user-attachments/assets/d87d37dc-fc17-410a-b015-be9bd97d13ad)
   - The search reveals that there was no login attempt made using the backdoor account
7.  To find the command that was used to add a backdoor user from a remote computer search using **index=main A1berto**. Then select the **CommandLine** field in the left pane to see the command that was used
   
    ![image](https://github.com/user-attachments/assets/daf43de1-b1d4-4c35-ae81-f91fd5b72c3b)
    ![image](https://github.com/user-attachments/assets/a30f96a7-a578-4278-a58f-4da6ce5ec6a9)
    - The first value is the correct command because this [reference](https://lolbas-project.github.io/) explains that the WMIC.exe is a command line that can execute binary on a remote system
8. Suspicious Powershell commands were executed, so look for the name of the infected host. There are two ways to do this:
   1. Search with the command **Powershell**, then select the **Hostname** field on the left pane to see the name of the host

      ![image](https://github.com/user-attachments/assets/e0cbe41e-6b70-4cb9-8dbd-8726d97ff396)
      ![image](https://github.com/user-attachments/assets/2a4e4df9-db40-44e8-af1e-62f9766fa721)
      - The host name is James.browne
   2. Search using **EventID=4103 OR EventID=4104** to find the host executing suspicious Powershell commands, then selecting the **Hostname** field on the left pane to see the host's name
      - Event ID 4103 (Module Logging) and Event ID 4104 (Powershell Script Block Logging) both log Powershell activities

      ![image](https://github.com/user-attachments/assets/0447c067-e055-4bec-a0a2-679d5c351b89)
      ![image](https://github.com/user-attachments/assets/bab5f278-b3eb-41c3-9831-ee128e43953f)
9. PowerShell logging was enabled on this device, so check how many events were logged for the malicious PowerShell execution by using **EventID=4103 OR EventID=4104** in the search
   
    ![image](https://github.com/user-attachments/assets/ef1d598d-be23-4052-aa53-6ab3b4c3b1e9)
   - 79 events were logged
10. One of the Powershell scripts from the infected host had initiated a web request. However the script is encoded, so need to decode it to find the full URL
    1. Using the previous search, look for the encoded script under **Host Application**. The script is Base64 encoded 
   
       ![image](https://github.com/user-attachments/assets/ef1d598d-be23-4052-aa53-6ab3b4c3b1e9)
       ![image](https://github.com/user-attachments/assets/e6bc3b5d-6d8f-4141-a5b5-91dd839f7fae)
    2. Copy and paste the encoded script into [Cyberchef](https://gchq.github.io/CyberChef/), a website that'll help decode it
   
       ![image](https://github.com/user-attachments/assets/ac256470-0be8-4807-a794-8f34401008b0)
    3. In the Operations section (left panel), drag and drop **From Base64** and **Decode text** selections to the Recipe section. Under the Decode text section, change the Encoding to **UTF-16LE (1200)** to see the decoded text
   
       ![image](https://github.com/user-attachments/assets/24cf787c-7782-445a-b7ee-1ee2395da322)
    4. In the output, there's another Base64 encoded string that must be decoded. Additionally, there's a /news.php that may be a subdirectory 
      
       ![image](https://github.com/user-attachments/assets/471e351c-d3a5-45b9-81b7-b0f4d2fdaf6f)
    5. Open another Cyberchef page with the same Recipe (**From Base64** and **Decode text: UTF-16LE (1200)**) and paste the second encoded string that was found to decode it
   
       ![image](https://github.com/user-attachments/assets/f1da7eee-860b-4857-ad46-95b47256ade5)
    6. To obtain the full URL, combine the second decoded value with the potential subdirectory. Defang the URL by placing the text in the Input section and change the Recipe to a **Defang URL** operation.
       - Defanging is a necessary step as the URL could be potentially harmful and defanging will modify the potentially harmful link, making the URL safe to share
   
       ![image](https://github.com/user-attachments/assets/356cbdc3-8dd3-478a-9e7a-466fd97c9856)
       - The full URL is hxxp[://]10[.]10[.]10[.]5/news[.]php





    


  

   
