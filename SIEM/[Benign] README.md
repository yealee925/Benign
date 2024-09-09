# Benign

## Objective
The Benign challenge used a controlled environment to analyze logs within a Security Information and Management (SIEM) system to search for data that indicated potential cyber attacks. The project helped enhance understanding of indicators of compromise, attack patterns, and LoLBins.

### Skills Learned
- Advanced understanding of SIEM concepts and practical application
- Proficiency in analyzing and interpreting event logs
- Development of critical thinking and problem-solving skills in cybersecurity

### Tools Used
- Splunk for log ingestion and analysis

## Scenario
One of the client’s IDS indicated a potentially suspicious process execution indicating one of the hosts from the HR department was compromised. Some tools related to network information gathering / scheduled tasks were executed which confirmed the suspicion. Due to limited resources, we could only pull the process execution logs with Event ID: 4688 and ingested them into Splunk with the index **win_eventlogs** for further investigation.

The network is divided into three logical segments. It will help in the investigation.
- IT Department
  - James
  - Moin
  - Katrina
- HR Department
  - Haroon
  - Chris
  - Diana
- Marketing Department
  - Bell
  - Amelia
  - Deepak

### Steps
1. Open Splunk as it will have collected event logs which will allow us to monitor, search, and analyze the aggregated data

   ![image](https://github.com/user-attachments/assets/dee6483b-7626-4116-8831-429cb3385af5)
2. On the left panel, select the "Search & Reporting" tab which will open a new window that will allow you to search through the logs

   ![image](https://github.com/user-attachments/assets/b1bef82d-5d56-4c4f-b55e-852adaf0c74e)
3. We will be focusing on the logs that were ingested from the month of March in 2022. Change the settings beside the Search menu to:
   - Smart Mode --> **Verbose Mode** (to see all events and field data)
   - Last 24 Hours --> **Date Range** > **Between** **03/01/2022** and **03/31/2022**

   ![image](https://github.com/user-attachments/assets/c9ce2777-c89f-4ef3-8a1b-bafdb83677ad)
4. Input **index=win_eventlogs** in the search field to see all the events

   ![image](https://github.com/user-attachments/assets/0972e45c-0b64-455d-bd55-91160dcb400b)
     - The results show there are 13,959 evemts that were collected during March of 2022
5. There appears to be an imposter account recorded in the logs. Find the name of all the Users by inputing **stats count by UserName**

   ![image](https://github.com/user-attachments/assets/cd5eae7e-36eb-4950-9163-fd68b1e5bc7a)
   ![image](https://github.com/user-attachments/assets/520cd4c5-b060-4306-8d59-c2add05015ae)
   - The imposter account is *Amel1a* who attempted to impersonate the user Amerlia
6. Check which user from the HR deparment was observed to to be running scheduled tasks by inputting either **schtasks** or **schtasks.exe**. Then select the UserName field in the left pane to see the users

   ![image](https://github.com/user-attachments/assets/123aee20-197c-46e2-8587-25e0a957d6a9)
   ![image](https://github.com/user-attachments/assets/5ef89ef5-3844-4847-b56f-87efce162e03)
   - Out of the four we're given, only one of the users is from the HR Department, so *Chris.fort* is the correct user in this situation

### Steps finding LoLBin
1. Next let's look into some information about a system process (LOLBin) that was executed to download a payload from a file-sharing host. It's indicated the user who executed the system process was from the HR department
2. First, let's find the name of the system process used to help find the user who executed the command. Use the HostName field and enter **HostName="HR***" in the search bar [or can use the UserName field and input **(UserName="haroon" OR "Chris.fort" OR "Daina")**]
  
   ![image](https://github.com/user-attachments/assets/520311aa-1fe6-48e1-b05e-9923babc822a)
   ![image](https://github.com/user-attachments/assets/a4a13db2-6671-4bc4-8af1-64dab510fb00)

4. Select the CommandLine field in the left pane and select **Rare Values** at the top which will open a new search. The search will open in the Visualization section, switch to the **Statistics** section
    
   ![image](https://github.com/user-attachments/assets/6e375f3c-77f1-4e9c-b978-2ec0fc366b1d)
   ![image](https://github.com/user-attachments/assets/eb5b1e42-282a-4e6e-bc1b-7e2a1ef32292)
5. Using this [list](https://lolbas-project.github.io/), search for a command that includes a binary found on the list
  
   ![image](https://github.com/user-attachments/assets/c682b113-88cd-478e-a3f5-502898c7dd5d)
   ![image](https://github.com/user-attachments/assets/430ac3e0-c45b-46c8-b44c-8d29aa7d5527)
   - The first command contains *certutil.exe* which functions include downloading files from the Internet
6. Now that the command was found, click on the command and options will appear. Select **View events** to open a new search that includes the Command in the search bar

   ![image](https://github.com/user-attachments/assets/b5321851-b292-40d0-bd84-0fe187fa660f)
   ![image](https://github.com/user-attachments/assets/911eeb26-f730-4d9d-ac8b-6d82e34f0ffa)
7. The results of the search shows more information on the event including the name of user who executed the command

   ![image](https://github.com/user-attachments/assets/595017b6-2f80-472f-8cc1-5c9e66ecc203)
   - The infected user who executed the command was *haroon*
 8. The information from the event log also reveals:
    - The command was executed on **03/04/2022**
    - The third-party site that was used to download the malicious payload is **controlc.com**
      - The URL that haroon (infected host) connected to was **https://controlc.com/e4d11035**
    - **benign.exe** was the file that was saved on the host machine from the C2 server

    ![image](https://github.com/user-attachments/assets/6c8efd76-a41e-4319-876a-cdf01767a602)




      

 

   

