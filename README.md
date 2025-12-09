# Section 1: Introduction
   Cybersecurity monitoring and incident response rely solely on the ability to detect, analyse, and interpret system and cloud activity. As new organisations increasingly adopt cloud services and distributed environments, analysts require tools capable of correlating large volumes of heterogeneous log data. Splunk plays an important role by providing a centralised platform for ingesting and analysing machine data to support threat hunting, breach analysis, and security decision making.
   This report documents an investigation carried out using the Splunk BOTSv3 dataset, a purpose-built training dataset commonly used for analyst development and cyber range exercises. The dataset simulates a real-world compromise within a hybrid enterprise environment, including AWS activity logs, endpoint telemetry, authentication records, and adversarial behaviour. The objective of this investigation was to identify key indicators of compromise (IOCs), answer guided forensic questions and reconstruct attacker behaviour based on observable evidence.

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



