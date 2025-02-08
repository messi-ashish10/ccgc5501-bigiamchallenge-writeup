# Challenge number
Challenge: 4

## Challenge Statement
This time I was supposed to interact with an S3 bucket which has been configured for public access. The task was to find vulnerabilities in the configuration and read a flag stored in a file in the bucket. My challenge would not be complete until I submitted the flag once I retrieved it.

## IAM Policy
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::thebigiamchallenge-admin-storage-abf1321/*"
        },
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::thebigiamchallenge-admin-storage-abf1321",
            "Condition": {
                "StringLike": {
                    "s3:prefix": "files/*"
                },
                "ForAllValues:StringLike": {
                    "aws:PrincipalArn": "arn:aws:iam::133713371337:user/admin"
                }
            }
        }
    ]
}

```
### Short Analysis about Challenge
**What do I have access to?**

I have public access to download objects from the S3 bucket due to the s3:GetObject permission. This means anyone, including unauthorized users, can retrieve files stored in the bucket.

**What don't I have access to?**

I do not have full control over the bucket. While I can list files under the files/ prefix, this action is restricted to the IAM principal arn:aws:iam::133713371337:user/admin. This means I cannot see the full list of objects unless I know their exact names.

**What do I find interesting?**

The fact that s3:GetObject is allowed for all users is a major security risk. This misconfiguration enables public access to sensitive files, making them accessible without authentication.

**What could an attacker do with this access?**

An attacker could exploit this misconfiguration to download and extract sensitive information from the bucket, including the flag required to complete the challenge. If the bucket contains confidential data, this could lead to a data breach.
  
**How could this misconfiguration have been avoided?**

A more secure setup would restrict the s3:GetObject permission to specific IAM users or roles instead of allowing public access. Using AWS Identity and Access Management (IAM) policies, organizations should enforce proper access controls and enable logging to monitor access patterns.


## Solution
**Step-1: List Available Files in the Bucket**

I attempted to list the objects within the files/ prefix using the following command:
```
aws s3 ls s3://thebigiamchallenge-admin-storage-abf1321/files/ --no-sign-request
```
**Step-2: Retrieve the Flag File and Extract the Flag**

I used the following command to download the flag-as-admin.txt file directly from the bucket:
```
aws s3 cp s3://thebigiamchallenge-admin-storage-abf1321/files/flag-as-admin.txt -  --no-sign-request
```
**Step 3: Extracted Flag**
```
{wiz:principal-arn-is-not-what-you-think}
```
**Step 4: Submit the Flag**

Finally, I submitted the retrieved flag on the challenge platform.

## Reflection
**What was your approach?**

I started by carefully examining the IAM policy to determine my level of access. Once I understood the permissions, I used the AWS CLI to attempt listing and retrieving objects from the S3 bucket. Since the bucket was misconfigured to allow public access, I was able to access the files without authentication.
  
**What was the biggest challenge?**

One of the main obstacles I faced was the restriction on listing objects within the bucket. Without the s3:ListBucket permission, I couldn't see all available files, which meant I had to find another way to locate the flag.
  
**How did you overcome the challenges?**

To work around this limitation, I experimented with different file names and leveraged the s3:GetObject permission, which was publicly accessible. By guessing potential filenames and attempting to download them, I was eventually able to retrieve the correct file.
  
**What led to the break through?**

The key moment came when I successfully downloaded the flag file. This confirmed that public access was enabled, proving that anyone could retrieve files from the bucket without authentication.
  
**On the blue side, how can the learning be used to properly defend the important assets?**

This challenge reinforced the importance of securing S3 buckets properly. To prevent unauthorized access, organizations should follow the principle of least privilege, ensuring that only specific users or roles have access. Regular security audits, proper IAM role configurations, and enabling encryption are essential steps to protect sensitive data from exposure.
