# Assigning IAM Users Permissions

We're going to use CloudFormation to apply a template, which will create the following Logical Resources:\
(1) IAM User ("Sally") with a managed policy attached requiring you to change the password upon first login. \
(2) S3 Buckets ("catpics"; "animalpics"). \
Policy Resource, which is a Managed Policy that allows access to all S3 Buckets, except for the "catpics" bucket!
If you'd like to perform these steps without CFN Automation, perform the steps in these manuals.

[Adding IAM Admin User](https://github.com/rmathisen-aws/Adding_IAM_Admin_User/blob/main/README.md) \
[S3 Bucket](https://github.com/rmathisen-aws/S3_Bucket)

CloudFormation → Create Stack \
Upload Template → Choose File → loacte the demo_cfn.yaml file & open \
Next

Stack Name: IAM \
Parameter: sallypassword (choose a PW) \
Next

Next

Because this Stack will create an IAM User, scroll all the way down and acknowledge this "Capability" \
Create Stack

Click on the Resources tab and see the (4) resources that have been created. \
S3 Buckets: animalpics & catpics \
IAM Managed Policy: policy \
IAM User: sally

Right Click on Sally's Physical ID & open in new tab \
Notice the attched policy IAMUserChangePassword

CloudFormation → Stacks → click on the IAM stack → Outputs tab → copy the Value for the Sally User Name \
Open a New Browser, or Private Tab \
Log into AWS console using the Sally User Name Value \
Now you'll be prompted to change the PW \
Notice that you don't have any permissions to access EC2, or S3, or any other permissions!

**Assign S3 Full Admin as an Inline Policy:** \
Copy the following into your clipboard: \
{
   "Version":"2012-10-17",
   "Statement":[
      {
         "Sid":"statement1",
         "Effect":"Allow",
         "Action":[
            "s3:*"
         ],
         "Resource":[
            "arn:aws:s3:::*"
         ]
       }
    ]
} \ 

Go back to where you're logged into your iamadmin \
IAM → Access Management → Users → click on the Sally User Name \
Permissions tab → Add Inline Policy → JSON tab & paste in the JSON policy \
Review Policy \
Name: s3admininline \
Create Policy

Notice the AWS Managed Policy and Inline Policy

\
Go back to where you're logged in as Sally \
S3 → Buckets \
Now we have access to iam-animalpics & iam-catpics Buckets

You will be able to upload files to these buckets, and open those files!
