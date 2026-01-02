# Section 1: Introduction
   Cybersecurity monitoring and incident response rely solely on the ability to detect, analyse, and interpret system and cloud activity. As new organisations increasingly adopt cloud services and distributed environments, analysts require tools capable of correlating large volumes of heterogeneous log data. Splunk plays an important role by providing a centralised platform for ingesting and analysing machine data to support threat hunting, breach analysis, and security decision making.
   This report documents an investigation carried out using the Splunk BOTSv3 dataset, a purpose-built training dataset commonly used for analyst development and cyber range exercises. The dataset simulates a real-world compromise within a hybrid enterprise environment, including AWS activity logs, endpoint telemetry, authentication records, and adversarial behaviour. 

# A Public GitHub Repository containing evidence, queries, and video link: 
 https://github.com/jcalamasonye/COMP3010_BOTSv3-Incident-Analysis.git

# Section 2: SOC Roles and Incident Handling Reflection
   A Security Operating System (SOC) serves as the main function for monitoring, analysing, and responding to security threats in an organisation. The concept of SOC has evolved over the past fifteen years as a strategic defence mechanism against increasingly sophisticated cyberattacks and operate using a tiered model that defines specific responsibilities based on experience, technical depth, and investigative authority [1]. The BOTSv3 investigation goes in connection with how the SOC tiers function in practice. 
  
### Tier 1 – Monitoring and Initial triage 
 Tier 1 plays a central role in identification and escalation of potential security incidents in a Security Operation Centre (SOC) such as monitoring SIEM dashboards, identifying alerts, and escalating potential evidence [2]. In BOTSv3, a Tier 1 analyst would recognise unusual AWS behaviour such as PutBucketAcl events, failed authentication attempts, or incorrect S3 reads and recognising anomalous endpoints via metadata commands such as | metadata type=hosts index=botsv3. Tier 1's main duty is observing deviations from baseline behaviours  

### Tier 2 – Investigation and Analysis 
   Tier 2 is known to perform stronger and better investigation, connecting information through cloud servers, endpoints, and identity sources [3]. During this task, the Tier 2 function is associated with performing the SPL queries, identifying the public exposure of the bucket frothlywebcode through S3 access logs, checking unexpected endpoint behaviour such as the operating system running Windows 10 Enterprise instead of Windows 10 Pro which was discovered using the following query:
 ### index=botsv3 sourcetype=WinHostMon Type=OperatingSystem 
Tier 2 analysis aim focuses on the understanding of the attack path not just identifying its presence. 


### Tier 3 – Threat Hunting and Specialist Analysis 
   Tier 3 analysts conduct advanced incident response, malware analysis, long-term threat hunting, and real-time identification of indicators of compromise (IOCs) such as shadow copy deletions, suspicious commands, and backup configuration modifications, enabling security teams to uncover adversarial behaviours before they disrupt recovery processes [4]. In BOTSv3, a Tier 3 analyst would reconstruct the entire attack chain. They would review persistence methods, analyse network links, switches across event types such as (AWS, windows security logs, hostmon telemetry), analyse MFA bypass and examine correlations that indicate advancement or authorised progress.
## 2.1.  Incident Handling Methodology Applied to BOTSv3
   The Incident follows a pattern, which are all discussed and highlighted for better understanding and to show how a SOC maintains forensic accuracy while escalating an incident from detection to response.
     During the BOTSv3 investigation, the preparation phase involves setting up Splunk, taking in all the data, and validating the sourcetypes. While detection and analysis were taking place, there was a suspicious CloudTrail event identified, unusual operating system versions are examined, and misconfigured S3 ACLs are observed. Containment focuses on determining compromised users or assets, isolating public S3 buckets [5], and preventing further data exposure and also the eradication phase includes the removal of unsafe permissions, enforcing MFA, correcting IAM roles, and fixing cloud policies, followed by recovery efforts that restore systems to secure configurations, verify logs, and ensure ACLs goes back to least-privilege settings and finally, post incident activity consists of analysing roots causes, keeping track of lessons learned and improving detective rules and alerts.  

   # Section 3: Installation and Data Preparation 
  The successful execution of the BOTSv3 investigation and installation required a correctly configured analysis environment, that includes a functional Ubuntu virtual on Oracle VirtualBox, a full installation of Splunk enterprise, ingestion of the BOTSv3 dataset, and verification that all log sources were done properly. 

  ## 3.1. Virtual Machine Setup 
  The investigation and practice were done on Oracle VirtualBox, selected due to its stability, compatibility and how suitable it is for isolated cybersecurity testing. According to the VirtualBox configuration screenshot displayed in Fig. 1, the hardware environment comprised of a Windows 11 host operating system running an Ubuntu 20.04 LTS virtual machine in VirtualBox with 8 GB of allotted memory, 60 GB of storage, and a secure, isolated NAT network mode.

  
  <img width="452" height="294" alt="image" src="https://github.com/user-attachments/assets/45eb1238-a4c7-40a6-a6f2-344ece3d16a2" />
   Fig 1. Oracle VirtualBox Environment 

   ## 3.2. Splunk Enterprise Installation on Ubuntu 
   After I installed and configured Ubuntu, I proceeded on downloading Splunk from the Splunk Enterprise from the official website on Ubuntu into the download’s directory. Below, figure 2 shows the downloaded .deb installer together with the BOTSv3 dataset. 


   <img width="452" height="281" alt="image" src="https://github.com/user-attachments/assets/a075eee3-11e1-47b4-92e2-69e789ffccaa" />
   Fig 2. Files in Ubuntu downloaded folder



   Splunk was installed using the package manager in terminal:
•	cd ~/Downloads
•	sudo dpkg -i splunk-10.0.1-<version>-linux-amd64.deb
After logging into Splunk Enterprise, the system wants user to accept the software license and create a login detail by running the following command 
•	sudo /opt/splunk/bin/splunk start --accept-license.
 After that, Splunk was configured to start automatically as a system service using 
•	sudo /opt/splunk/bin/splunk enable boot-start
 This ensure that it would start whenever the virtual machine comes up. Once the system starts running, the Splunk web interface became accessible through http://127.0.0.1:8000/,  which serves as a platform for log ingestion, indexing, and analytics throughout the project. Figure 3 and 4 shows the Splunk login screen displayed in the browser, showing that the installation was successful and the Splunk Enterprise is operational. 


 <img width="452" height="284" alt="image" src="https://github.com/user-attachments/assets/243d91d3-9643-4417-b4db-79b094e35a0c" />
Fig 3. Splunk Enterprise Login Screen


<img width="452" height="283" alt="image" src="https://github.com/user-attachments/assets/dd754820-d4e1-4471-834e-9ee022622793" />
Fig 4. Splunk Dashboard Confirming Successful Installation 


## 3.3.  BOTSv3 Dataset Ingestion 
   The BREACH and Outage Testing Scenario v3 (BOTSv3) dataset was gotten from Splunk's official GitHub repository. The compressed archive file (botsv3_data_set.tgz) was saved in Ubuntu Downloads directory as shown in the figures above. 
  To ingest this dataset, I used the Splunk approved deployment method which involves extracting the dataset directly into Splunk application directory with the following commands:
•	sudo tar zxvf botsv3_data_set.tgz -C /opt/splunk/etc/apps/
  This command automatically loads all pre-indexed event data into the botsv3 index. After extraction, I had to restart Splunk to make sure the dataset was fully activated. Ran a validation search to confirm the successful ingestion below. 


<img width="452" height="286" alt="image" src="https://github.com/user-attachments/assets/38762cb5-2ca1-4411-8ed5-bb5416cd0333" />
Fig 5. Event time


  With the above figure, that show a continuous pattern across 2018-2019, tells that more than 1 million events were indexed to confirm the successful data ingestion.
  
## 3.4.  Identification of Host in the Dataset
   Ran the following query in the figure below to determine the structure of the simulated enterprise network. 



<img width="452" height="287" alt="image" src="https://github.com/user-attachments/assets/fe0672c3-04e5-4959-8b80-43536eee1fad" />
Fig 6. Host in the Environment 


These revealed 30 distinct hosts, representing a realistic environment all in the figure above. 

## 3.5. Identifying Available Sourcetypes 
  To register the differences, the sourcetype metadata in the figure 7 was used. 

  <img width="452" height="292" alt="image" src="https://github.com/user-attachments/assets/1c321cf3-1c53-444d-9f41-e41557be5bc5" />
  Fig 7. Sourcetypes available 

  This shows that the dataset has over 107 sourcetypes, which are in figure 7 above. 

  
## 3.6. Validating Security-Relevant Event Sources
   Source types such as Windows security logs were used and this show logs containing event codes, event types which are required for attacking attacker’s responses. 


<img width="452" height="289" alt="image" src="https://github.com/user-attachments/assets/3e2880a7-a0e2-47f5-98c1-ed025b9c07b2" />
Fig 8. Security Relevant Windows event


## 3.7. Summary of Installation and Preparation 
  Going through figure 1 - 8, the screenshots document a standard technical workflow used to establish the Splunk investigation environment. this includes Creating and configuring an Ubuntu VirtualBox virtual machine, Installing Splunk Enterprise using the .db deployment method, ingesting the BOTSv3 dataset into Splunk, which helped create the bases for the investigative and threat hunting tasks and to also help better understand the process of splunk. 


# Section 4: Guided Questions and Analysis
 This section was carried out by answering all the 200-level BOTSv3 questions, the SPL query used, the results gotten, the screenshots taken, and its relevance with an SOC investigation. 

## 4.1. Q1 – IAM Users Accessed AWS Services
QUERY –  index=botsv3 sourcetype="aws:cloudtrail" | stats values(userIdentity.userName)


<img width="452" height="285" alt="image" src="https://github.com/user-attachments/assets/847a6e09-68b0-4561-bd97-5dd6e0232bf6" />
Fig 9. IAM User Accessed Service 


SOC Relevance – Getting an inadept on what identities accessed AWS services helps analysts profile behaviour, detect unauthorised access, and review privilege boundaries [6] . Multiple active IAM identities indicate a wide distribution of access, which increases the attack surface.

## 4.2. Q2 – Field Used to Detect MFA Absence
QUERY – index=botsv3 sourcetype=”aws:cloudtrail” |  stats count by eventName 

<img width="452" height="282" alt="image" src="https://github.com/user-attachments/assets/f9e3ac0c-efd1-4252-bf1e-8156ef7d7ddf" />
Fig 10. Field to Detect MFA

SOC Relevance – Every incident that was evaluated turned out to be false, indicating that AWS API actions were carried out without MFA and creating a serious cloud security issue [7]. 

## 4.3. Q3 – Processor Number Used on Web Server 
QUERY – index=botsv3 sourcetype=hardware | CPU_TYPE

<img width="452" height="278" alt="image" src="https://github.com/user-attachments/assets/4b90e05d-ce64-4028-8d1a-339f787f9637" />
Fig 11. Processor number 

SOC Relevance – Processor identification helps identify unauthorised devices, verify hardware data, and evaluate specifications for analytical responses [8].

## 4.4. Q4 – Event ID For Public S3 Bucket ACL Change 
QUERY – index=botsv3 sourcetype="aws:cloudtrail" eventName=PutBucketAcl | table eventID eventName requestParameters.bucketName 

<img width="452" height="279" alt="image" src="https://github.com/user-attachments/assets/24c30818-6fcd-4600-a069-00331685ae9b" />
Fig 12. Event ID 

SOC Relevance – The main reason for the S3 bucket's public exposure is this analysis. This study allowed SOC analysts to identify who approved the expose.

## 4.5. Q5 – Bud’s Username
QUERY – index=botsv3 sourcetype=”aws:cloudtrail” | stats values(userIdentity.userName)

<img width="452" height="283" alt="image" src="https://github.com/user-attachments/assets/1377dd37-eba5-47cd-b94e-215515a96605" />
Fig 13. Bud’s Username 

SOC Relevance – One of the main responsibilities of SOC is to determine who oversees unsafe configuration changes.

## 4.6. Q6 – Name of the S3 Bucket Made Public 
QUERY – index=botsv3 sourcetype=”aws:cloudtrail” eventName=PutBucketAcl | table requestParameters.bucketName userIdentity.userName

<img width="452" height="285" alt="image" src="https://github.com/user-attachments/assets/56906aee-06ff-403d-b073-8397ac801f38" />
Fig 14. Name of S3 Bucket

SOC Relevance – This shows which asset is affected by the misconfiguration. Analysts may assess exposure by knowing the bucket.

## 4.7. Q7 – Text File uploaded to the Public S3 Bucket
QUERY - index=botsv3 sourcetype="aws:s3:accesslogs" "*frothlywebcode*" "*.txt"

<img width="452" height="288" alt="image" src="https://github.com/user-attachments/assets/2a12c215-bb06-4b46-869c-79e790060020" />
Fig 15. Bucket File Name

SOC Relevance – The file name suggests to ethical disclosure, simulating a situation in which researchers alert businesses by placing harmless information in public storage.

## 4.8. Q8 – FQDN of Endpoint Running a Different Windows OS Edition
QUERY –  index=botsv3 sourcetype=winhostmon Type=OperatingSystem | stats values(OS) values(fqdn) by host

<img width="452" height="291" alt="image" src="https://github.com/user-attachments/assets/8f1f5f28-d3ca-4983-8059-e4fdf187f722" />
Fig 16. FQDN Endpoint

Answer – During my investigation, I figured out that only one endpoint used the windows 10 Enterprise which is BTOLL-L, further host investigation into BSTOLL-L revealed its full entity domain as BSTOLL-L.froth.ly. 

SOC Relevance – A unique OS version may highlight sensitive systems, testing or development settings, indications of tampering or rebuild, prompting SOC analyst to flag off these systems for more investigation. 


# Section 5: Conclusion and Recommendations 

 At the end of this report, the investigation shows how we can use Splunk to construct a good incident analysis across cloud, endpoint, and user activity logs within the enterprise environment [9] . Few problems were identified while using the BOTSv3 dataset, problems such as misconfigured S3 permissions and repeated activity linked to the IAM user bstoll, whose actions resulted in bad ACL changes, file access attempts and related endpoint behaviour. The findings of the publicly accessible frothlywebcode bucket and the uploaded " OPEN_BUCKET_PLEASE_FIX.txt" file shows how simple configuration errors can create opportunities for exploitation. 

 
 ## Recommendations 
  To improve the organisation's security standards, high quality measures should be improvised which are. IAM controls should be modified by enforcing least privilege access [10] , restricting high risk actions such as S3 ACL modifications and mandating MFA for all users. 
  S3 configuration must be thorough using block public access, continuous monitoring rules, and automated policy enforcement to prevent exploitation [11].
  Monitoring and detection functions should be improved by allowing real alerts on CloudTrail and S3 activity, especially in an unauthorised access attempt [12]
    Endpoint management needs to be of standards to that all workstations follow consistent operating system versions [13] , making deviations identifiable.
    Finally, staff should receive certain training on secure cloud practices, importance of MFA, and the risk associated with improper configuration changes [14]. 

    
## Final Reflection 
  BOTSv3 effectively replicated the challenges faced on modern social analysts, requiring analysts to integrate multiple data sources, critical thinking, and root cause analysis [15] . This investigation emphasised on the benefits of proactive monitoring, consistent operational standards, and connections between technical and organisational analysts to maintain good security practice [16]. 

  # Section 6: AI Use Declaration 

   Generative partnered AI tools were used in this course work. For the BOTSv3 investigation, ChatGPT was used for better execution such as planning and structuring, technical guidance and troubleshooting, and better research pattern. 



# References

[1] 	Y. Baddi, M. A. Almaiah, O. Almomani and Y. Maleh, The Art of Cyber Defense: From Risk Assessment to Threat Intelligence, CRC press, 2024, p. pp271.

[2] 	L. Kersten, S. Darré, T. Mulders, E. Zambon, M. Caselli and C. Snijders, “A Security Alert Investigation Tool Supporting Tier 1 Analysts in Contextualizing and Understanding Network Security Events,” 2024 Annual Computer Security Applications Conference (ACSAC), pp. pp. 890-905, 09-13 December 2024.

[3] 	A. L. Bruhn, K. L. Lane and S. E. Hirsch, “A Review of Tier 2 Interventions Conducted Within Multitiered Models of Behavioral Prevention,” vol. 22, no. 3, 4 March 2013.

[4] 	N. Alsharabi, A. Bhardwaj, A. Ayaba and A. Jadi, “Threat hunting for adversary impact inhibiting system recovery,” Computers & Security, vol. 154, p. pp. 104532, September 2025. 

[5] 	E. Al-Dahasi and F. A. Khan, “Automating Security Incident Response in SCADA Systems through SIEM-ML Integration,” 2024 29th International Conference on Automation and Computing (ICAC), pp. pp. 1-10, August 2024. 

[6] 	D. N. Srisai and D. K. Rattanapong, “Securing AWS Resources with IAM: Identity-Based Policies, Actions, and Permissions Flow,” Vols. 9, no.4, no. pp. 14-22, 2022. 

[7] 	E. Caffagni, D. W. Eaton, J. P. Jones and M. v. d. Baan, “Detection and analysis of microseismic events using a Matched Filtering Algorithm (MFA),” Geophysical Journal International, vol. 206, no. 1, pp. pp. 644-658, 03 April 2016. 

[8] 	X. Su, C.-C. Chu, B. S. Prabhu and R. Gadh, “On the Identification Device Management and Data Capture via WinRFID1 Edge-Server,” vol. 1, no. 2, pp. pp. 95-104, 31 December 2007. 

[9] 	P. Shelke and T. Frantti, “Exploring the Possibilities of Splunk Enterprise Security in Advanced Cyber Threat Detection,” Proceedings of the 20th International Conference on Cyber Warfare and Security (ICCWS 2025) , vol. 20, no. 1, pp. pp. 605-613, 2025. 

[10] 	S. Vitla, “Securing the physical and digital frontier: leveraging identity and access management (IAM) to address the lack of controls on physical access to sensitive systems,” Cyber Risk Security and Governance, Cotelligent India Pvt Ltd (A TechDemocracy Company), Hyderabad, Telangana, vol. 06, no. 02, pp. pp. 108-125, 07 July 2022. 

[11] 	F. Siddiqui, M. Hagan and S. Sezer, “Embedded Policing and Policy Enforcement Approach for Future Secure IoT Technologies,” Living in the Internet of Things: Cybersecurity of the IoT, 2018. 

[12] 	D. Tykholaz, R. Banakh, L. Mychuda, A. Piskozub and Kyrychok, “Incident response with AWS detective controls,” Cybersecurity Providing in Information and Telecommunication Systems, vol. 3826, pp. PP. 190-197, 06 Decemeber 2024. 

[13] 	S. Mistry, P. Lalwani and M. B. Potdar, “Endpoint Protection through Windows Operating System Hardening,” International Journal of Computer Applications Technology and Research, vol. 7, no. 02, 2018. 

[14] 	S. H. Kendyala, “THE ROLE OF MULTI FACTOR AUTHENTICATION IN SECURING CLOUD BASED ENTERPRISE APPLICATIONS,” p. pp. 16, 11 November 2020. 

[15] 	A. T. Chen, M. Komi, S. Bessler, S. P. Mikles and Y. Zhang, “Integrating statistical and visual analytic methods for bot identification of health-related survey data,” Journal of Biomedical Informatics, vol. 104439, p. pp.144, August 2023. 

[16] 	C. Onwubiko, “Cyber security operations centre: Security monitoring for protecting business and supporting cyber defense strategy,” 2015 International Conference on Cyber Situational Awareness, Data Analytics and Assessment (CyberSA), pp. pp. 1-10, 27 July 2015. 

[17] 	F. Siddiqui, M. Hagan and S. Sezer, “Embedded Policing and Policy Enforcement Approach for Future Secure IoT Technologies,” Living in the Internet of Things: Cybersecurity of the IoT, 2018. 

[18] 	D. Tykholaz, R. Banakh, L. Mychuda, A. Piskozub and Kyrychok, “ncident response with AWS detective controls Cybersecurity Providing in Information and Telecommunication Systems,” vol. 3826, pp. pp. 190-197, December 06 2024. 

[19] 	S. Mistry, P. Lalwani and M. B. Potdar, “Endpoint Protection through Windows Operating System Hardening,” International Journal of Computer Applications Technology and Research, vol. 7, no. 2, pp. pp. 58-62, 2018. 






# Student Declaration of AI Tool use in this Assessment
Please indicate your level of usage of generative AI for this assessment - please tick the appropriate category(s). 
If the “Assisted Work” or “Partnered Work” category is selected, please expand on the usage and in which elements of the assignment the usage refers to.

## Solo Work	S1 - Generative AI tools have not been used for this assessment.	☐

## Assisted Work	A1 – Idea Generation and Problem Exploration
Used to generate project ideas, explore different approaches to solving a problem, or suggest features for software or systems. Students must critically assess AI-generated suggestions and ensure their own intellectual contributions are central.	☐

### A2 - Planning & Structuring Projects
AI may help outline the structure of reports, documentation and projects. The final structure and implementation must be the student’s own work.	☐

### A3 – Code Architecture
AI tools maybe used to help outline code architecture (e.g. suggesting class hierarchies or module breakdowns). The final code structure must be the student’s own work.	☐

### A4 – Research Assistance
Used to locate and summarise relevant articles, academic papers, technical documentation, or online resources (e.g. Stack Overflow, GitHub discussions. The interpretation and integration of research into the assignment remain the student’s responsibility.	☐

### A5 - Language Refinement
Used to check grammar, refine language, improve sentence structure in documentation not code. AI should be used only to provide suggestions for improvement. Students must ensure that the documentation accurately reflects the code and is technically correct.	☐

### A6 – Code Review
AI tools can be used to check comments within the code and to suggest improvements to code readability, structure or syntax.  AI should be used only to provide suggestions for improvement. Students must ensure that the code accurately reflects their knowledge and is technically correct.	☐

### A7 - Code Generation for Learning Purposes
Used to generate example code snippets to understand syntax, explore alternative implementations, or learn new programming paradigms. Students must not submit AI-generated code as their own and must be able to explain how it works.	☐

### A8 - Technical Guidance & Debugging Support
AI tools can be used to explain algorithms, programming concepts, or debugging strategies. Students may also help interpret error messages or suggest possible fixes. However, students must write, test, and debug their own code independently and understand all solutions submitted.	☐

### A9 - Testing and Validation Support
AI may assist in generating test cases, validating outputs, or suggesting edge cases for software testing. Students are responsible for designing comprehensive test plans and interpreting test results.	☐

### A10 - Data Analysis and Visualization Guidance
AI tools can help suggest ways to analyse datasets or visualize results (e.g. recommending chart types or statistical methods). Students must perform the analysis themselves and understand the implications of the results.	☐

### A11 - Other uses not listed above
Please specify:	☐

## Partnered Work	P1 - Generative AI tool usage has been used integrally for this assessment
Students can adopt approaches that are compliant with instructions in the assessment brief.
Please Specify: Partnered use of AI such as ChatGPT (OpenAI) to help with researching, report structuring, installation and preparation and troubleshooting splunk queries which are in alignment with the course work brief 	☒


## Please provide details of AI usage and which elements of the coursework this relates to:

   In this coursework, I used ChatGPT in the partnered role to fully understand the background concepts such as SOC tiers, Splunk and incident handling, to also help plan a better structure of my BOTSv3 incident report and GITHUB README, to better improve my wording and grammar usage, and to get better ideas for troubleshooting Splunk queries. The final searches, analysis, quiz answers, video content and conclusion are all my work 

### I understand that the ownership and responsibility for the academic integrity of this submitted assessment falls with me, the student.	☒

### I confirm that all details provide above are an accurate description of how AI was used for this assessment.	☒



















                                               





