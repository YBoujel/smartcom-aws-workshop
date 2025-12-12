# smartcom-aws-workshop
hands on on aws basic services
Cloud Computing Basics: Hands-on Lab Guide

Instructor: Yassine Boujelben | Event: SMARTCOM Workshop

🛠️ Prerequisites & Setup

Target Environment: AWS Sandbox (e.g., AWS Academy Learner Lab) or a Free Tier AWS Account.

Before you begin:

Log in to your AWS Console.

Ensure you are in the us-east-1 (N. Virginia) region (standard for labs to ensure service availability).

For Lab 4, you will need the AWS CLI installed on your local machine, or you can use the AWS CloudShell directly in the browser (Recommended for beginners).

🧪 Lab 1: Infrastructure as a Service (EC2)

Objective: Launch a virtual server, configure security, and access it via SSH and HTTP.

Steps:

Navigate to EC2:

In the AWS Console search bar, type EC2 and select it.

Click the orange Launch Instance button.

Name & OS:

Name: Web-Server-Lab

AMI (Amazon Machine Image): Select Amazon Linux 2023 AMI (Free tier eligible).

Instance Type:

Select t2.micro (1 vCPU, 1 GiB Memory).

Key Pair (Your "Keys" to the Server):

Click Create new key pair.

Key pair name: smartcom-key.

Key pair type: RSA.

Private key file format: .pem (OpenSSH).

Click Create key pair.

⚠️ Important: The file smartcom-key.pem will download to your computer. Do not lose this file. You cannot download it again.

Network Settings (The Firewall Experiment):

Click Edit next to Network settings.

Ensure "Auto-assign public IP" is Enable.

Select Create security group.

Security group name: Web-SG-Experiment.

Description: Testing security rules.

🛑 Critical Step: Remove any default Inbound rules if they exist (click "Remove"). We want the Security Group to be empty for now.

User Data (The Automation Magic):

Scroll down to Advanced details.

Scroll to the very bottom to the User data box.

Paste the following script. This script runs once when the server starts.

#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello SMARTCOM! This is my first Cloud Server.</h1>" > /var/www/html/index.html


Launch:

Click Launch Instance.

Click View all instances. Wait for the Instance State to turn Running.

🕵️ Part A: The "Broken" Server (Security Groups)

Select your instance.

Copy the Public IPv4 address.

Paste it into a browser tab (http://YOUR-IP).

Observation: The site will spin and eventually time out.

Why? AWS Security Groups (Firewalls) deny all inbound traffic by default unless explicitly allowed.

🛠️ Part B: Fixing the Firewall

Go back to the EC2 Console.

With your instance selected, click the Security tab (bottom pane).

Click the link under Security groups (e.g., sg-0123...).

Click Edit inbound rules.

Add Rule 1 (Web):

Type: HTTP | Source: Anywhere-IPv4 (0.0.0.0/0).

Add Rule 2 (SSH):

Type: SSH | Source: Anywhere-IPv4 (Or "My IP" for better security).

Click Save rules.

Refresh your browser tab. Success! You should now see the "Hello SMARTCOM" message.

💻 Part C: SSH Access

Now, let's log inside the server terminal.

For Mac/Linux Users (Terminal):

Open your terminal.

Navigate to where you downloaded the key (e.g., cd Downloads).

Set permissions (Critical step!):

chmod 400 smartcom-key.pem


Connect:

# Replace 1.2.3.4 with your Instance Public IP
ssh -i smartcom-key.pem ec2-user@1.2.3.4


For Windows Users (PowerShell):

Open PowerShell.

Navigate to your downloads folder (cd Downloads).

Connect:

ssh -i smartcom-key.pem ec2-user@1.2.3.4


(Note: If you get a "Permissions" error, right-click the .pem file -> Properties -> Security -> Advanced -> Disable Inheritance -> Remove all users except yourself).

Once Connected:
Type whoami to verify you are ec2-user. You are now controlling a computer in a Data Center in Virginia!

🧪 Lab 2: Monitoring & Metrics (CloudWatch)

Objective: Observe the "health" of the server we just built.

Steps:

Generate Load (Optional):

If you just launched the instance, CPU usage will be low. To see a spike, you would typically SSH in and run a stress test, but for now, we will just look at the idle metrics.

View Metrics:

Go back to your EC2 Dashboard.

Select your Web-Server-Lab instance.

Click the Monitoring tab in the bottom details pane.

Analyze:

Look at the CPU Utilization graph.

Look at Network In and Network Out.

Discussion Point: In a physical data center, getting these graphs requires installing agent software and setting up a monitoring server. In AWS, it comes free out of the box.

🧪 Lab 3: Static Website Hosting (S3)

Objective: Host a serverless website using Object Storage.

Steps:

Create a Bucket:

Navigate to S3.

Click Create bucket.

Bucket name: smartcom-lab-yourname-123 (Must be globally unique!).

Region: us-east-1.

Public Access Settings (Crucial):

Uncheck Block all public access.

Check the box: "I acknowledge that the current settings might result in this bucket and the objects within becoming public."

Click Create bucket.

Upload Content:

Create a file named index.html on your computer with some text (e.g., "Welcome to my S3 Site").

Open your bucket in AWS.

Click Upload -> Add files -> Select your index.html -> Upload.

Enable Hosting:

Go to the Properties tab of your bucket.

Scroll to the bottom: Static website hosting.

Click Edit -> Enable.

Index document: index.html.

Click Save changes.

Bucket Policy (Permissions):

Go to the Permissions tab.

Scroll to Bucket policy -> Edit.

Paste the following JSON (Replace YOUR-BUCKET-NAME with your actual bucket name):

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::YOUR-BUCKET-NAME/*"
        }
    ]
}


Click Save changes.

✅ Verification:

Go back to the Properties tab.

Scroll down to Static website hosting.

Click the Bucket website endpoint link.

Success: Your HTML file should load in the browser.

🧪 Lab 4: Data Protection with Versioning (S3)

Objective: Demonstrating how to recover accidental deletions or overwrites.

Steps:

Enable Versioning:

Use the same bucket from Lab 3.

Go to Properties.

Under Bucket Versioning, click Edit -> Enable -> Save changes.

The Overwrite Scenario:

On your computer, create a text file secret.txt containing text: "Version 1: The password is 1234".

Upload it to S3.

Edit the file on your computer to say: "Version 2: The password is CHANGED".

Upload it again to S3 (same name).

View Versions:

In the Objects tab, you will see only one secret.txt.

Toggle the switch Show versions (usually near the top of the list).

You will now see both versions of the file securely stored.

The Deletion Scenario:

Select secret.txt (the current version) and click Delete.

Type delete to confirm.

Toggle Show versions again. You will see a Delete Marker on top. The data is still there!

To Restore: Select the Delete Marker and delete it. The original file will reappear.

🧪 Lab 5: Function as a Service (AWS Lambda)

Objective: Run code without provisioning any servers.

Steps:

Create Function:

Navigate to Lambda.

Click Create function.

Function name: HelloSmartcomFunction.

Runtime: Python 3.x (latest).

Click Create function.

Add Code:

Scroll down to the Code Source editor.

Replace the default code with:

import json

def lambda_handler(event, context):
    print("I am running in the cloud!")
    return {
        'statusCode': 200,
        'body': json.dumps('Hello from AWS Lambda! No servers here!')
    }


Deploy & Test:

Click Deploy (Grey button) to save changes.

Click Test (Blue button).

Create a new test event (Name: Test1), leave defaults, click Save.

Click Test again.

✅ Verification:

You should see a green box: Execution result: succeeded.

Expand the Details to see your "Hello from AWS Lambda" message.

Note: You did not choose an OS, a CPU size, or setup networking. AWS handled it all.

🧪 Lab 6: Introduction to Automation (AWS CLI)

Objective: Control the cloud using the Command Line Interface instead of the mouse (Infrastructure as Code basics).

Setup:

Click the CloudShell icon ( >_ ) in the top navigation bar of the AWS Console.

Wait for the terminal to prepare.

Commands to Try:

1. Verification:
Check who you are logged in as.

aws sts get-caller-identity


2. List Storage:
List all buckets in your account.

aws s3 ls


3. Create Resources via Code:
Create a new bucket directly from the terminal.

# Syntax: aws s3 mb s3://<unique-name>
aws s3 mb s3://smartcom-cli-test-yourname


4. Upload Data via Code:
Create a file and upload it.

echo "This file was created by a robot" > bot.txt
aws s3 cp bot.txt s3://smartcom-cli-test-yourname/


5. Clean Up:
Delete the bucket and file.

aws s3 rb s3://smartcom-cli-test-yourname --force


🏆 The "Final Boss" Challenge

Scenario: A client needs a backup storage solution, but they are paranoid about losing data.
Time Limit: 15 Minutes.
Constraint: Do not look at previous lab instructions.

Create a New S3 Bucket with a unique name.

Enable Versioning on it.

Upload a file named important-data.txt.

Delete the file.

Recover the file.

Bonus: Can you find the S3 region code via the CLI?

🧹 Final Cleanup (Important!)

To avoid unexpected charges or running out of sandbox credits:

EC2: Select your instance -> Instance State -> Terminate.

S3: Select your buckets -> Empty -> Then Delete.

Happy Cloud Computing!
