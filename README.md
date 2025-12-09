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
