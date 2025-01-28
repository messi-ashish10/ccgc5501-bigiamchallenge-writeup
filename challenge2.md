# Challenge number
Challenge: 2

## Challenge Statement
In this challenge, I was tasked with interacting with an Amazon SQS queue that has been configured with public access, granting permissions to send and receive messages. The goal was to identify any security risks in the configuration and retrieve a flag from a message within the queue. Once the flag was retrieved, I was required to submit it to complete the challenge.

## IAM Policy
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": [
                "sqs:SendMessage",
                "sqs:ReceiveMessage"
            ],
            "Resource": "arn:aws:sqs:us-east-1:092297851374:wiz-tbic-analytics-sqs-queue-ca7a1b2"
        }
    ]
}
```
### write a short analysis about the IAM policy here
* What do I have access to?
-->I have public access to both send and receive messages from the specified SQS queue. This means that anyone, including unauthorized users, can interact with the queue by pushing or pulling messages without needing any credentials or authentication.

* What don't I have access to?
--> I cannot modify or delete messages once they are in the queue. My permissions are limited to sending and receiving messages, but I can't perform administrative tasks like altering or managing the messages.

* What do I find interesting?
--> The fact that the queue allows public access to both send and receive messages is concerning. It’s a major security risk, as it opens the queue to anyone, meaning sensitive data could be exposed, or attackers could misuse the service by flooding it with unwanted messages.

* What could an attacker do with this access?
--> An attacker could take advantage of this by sending malicious messages to the queue, possibly triggering unintended behaviors or disruptions. More importantly, they could receive messages containing sensitive data, such as the flag required to complete the challenge, which could lead to a data breach.
  
* How could this misconfiguration have been avoided?
--> The best approach would be to restrict the permissions so only authorized users or services can interact with the queue. A more secure setup would limit access using identity-based policies, ensuring that only specific accounts can send or receive messages. Additionally, encryption could be implemented to protect sensitive data stored in the queue.


## Solution
Step 1: Retrieve the Message from the SQS Queue: I used the following AWS CLI command to receive the message from the SQS queue. The command retrieves the latest message from the queue, which contains the flag's URL.
aws sqs receive-message --queue-url https://sqs.us-east-1.amazonaws.com/092297851374/wiz-tbic-analytics-sqs-queue-ca7a1b2 --attribute-names All --message-attribute-names All --no-sign-request

Step 2: Examine the Message: The returned message body contains a URL pointing to an S3 bucket:
{
    "URL": "https://tbic-wiz-analytics-bucket-b44867f.s3.amazonaws.com/pAXCWLa6ql.html",
    "User-Agent": "Lynx/2.5329.3258dev.35046 libwww-FM/2.14 SSL-MM/1.4.3714",
    "IsAdmin": true
}

Step 3: Access the URL in the Message: The message body provided a direct URL to an HTML file stored in an S3 bucket. I accessed the URL to view the file.
https://tbic-wiz-analytics-bucket-b44867f.s3.amazonaws.com/pAXCWLa6ql.html

Step 4: Inspect the HTML File for the Flag: I opened it in a web browser. Inside the HTML content, I found the flag.
{wiz:you-are-at-the-front-of-the-queue}

Step 5: Submit the Flag: Finally, I submitted the retrieved flag on the challenge platform.

## Reflection
* What was your approach?
--> My approach was straightforward. First, I analyzed the IAM policy to fully understand what actions I could perform. Then, I used the AWS CLI to interact with the SQS queue and test the commands to retrieve the messages. After receiving the messages, I sifted through them to find the flag.
  
* What was the biggest challenge?
--> The biggest challenge was understanding the exact scope of the IAM policy and ensuring I had the right permissions to receive messages. Additionally, confirming that the receive-message command would work as expected and return the correct data took a bit of trial and error.
  
* How did you overcome the challenges?
--> I overcame this challenge by carefully reviewing the IAM policy and testing different commands to verify what I had access to. The AWS CLI helped me check my assumptions and interact with the queue as needed.
  
* What led to the break through?
--> The breakthrough came when I successfully used the receive-message command. This allowed me to pull messages from the queue and find the message that contained the flag.
  
* On the blue side, how can the learning be used to properly defend the important assets?
-->This experience taught me the importance of the principle of least privilege. By limiting public access and using more restrictive IAM policies, organizations can protect critical assets like SQS queues. It's also vital to regularly audit and review the permissions of resources to avoid these types of security risks.
