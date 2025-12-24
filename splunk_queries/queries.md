# Splunk Queries used for 200-level BOTSv3 Investigation 

## This Document have all the Splunk queries used to answer the 200-level BOTSv3 questions (Q1-Q8).
## Each of the queries is labeled and well documented for clarity and to help construct the 2000 words assessment. 

# Q1 - IAM users that accessed the an AWS service 
## Query - index=botsv3 sourcetype="aws:cloudtrail" | stats values(userIdentity.userName)

# Q2 - Using CloudTrail to Detect AWS API Activity Without MFA
## Query - index=botsv3 sourcetype="aws:cloudtrail" | stats count by eventName

# Q3 - Processor number used on the web servers
## Query - index=botsv3 sourcetype=hardware | table CPU_TYPE

# Q4 - Event ID of the API call that made the S3 bucket public 
## Query - index=botsv3 sourcetype="aws:cloudtrail" eventName=PutBucketAcl | table eventID eventName requestParameters.bucketName

# Q5 - bud's Username
## Query - index=botsv3 sourcetype="aws:cloudtrail" | stats values(userIdentity.userName)

# Q6 - Name of the S3 bucket that was made public
## Query - index=botsv3 sourcetype="aws:cloudtrail" eventName=PutBucketAcl | table requestParameters.bucketName userIdentity.userName

# Q7 - Text file uploaded to the public S3 bucket
## Query - index=botsv3 sourcetype="aws:s3:accesslogs" "*frothlywebcode*" "*.txt"

# Q8 - FQDN of endpoint running a different Windows OS edition
## Query - index=botsv3 sourcetype=winhostmon Type=OperatingSystem | stats values(OS) values(fqdn) by host
## (Then isolate the host showing **Microsft Windows 10 Enterprise** which is **BSTOLL-L** 
