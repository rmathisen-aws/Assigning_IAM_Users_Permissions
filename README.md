# Assigning IAM Users Permissions

We're going to use CloudFormation to apply a template, which will create the following Logical Resources:\
(1) IAM User ("Sally") with a managed policy attached requiring you to change the password upon first login. \
(2) S3 Buckets ("catpics"; "animalpics"). \
Policy Resource, which is a Managed Policy that allows access to all S3 Buckets, except for the "catpics" bucket!

\
**Creating a Stack using CloudFormation Template:** \
CloudFormation → Create Stack \
Upload Template → Choose File → Copy & Paste in the following template \
Next

```
AWSTemplateFormatVersion: "2010-09-09"
Description: >
  This template implements an IAM user 'Sally'
  An S3 bucket for cat pictues
  An S3 bucket for dog pictures
  An S3 bucket for other animals
  And permissions appropriate for Sally.
Parameters:
  sallypassword:
    NoEcho: true
    Description: IAM User Sallys Password
    Type: String
Resources:
  catpics:
    Type:  AWS::S3::Bucket
  animalpics:
    Type:  AWS::S3::Bucket
  sally:
    Type: AWS::IAM::User
    Properties:
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/IAMUserChangePassword
      LoginProfile:
        Password: !Ref sallypassword
        PasswordResetRequired: "true"
  policy:
    Type: AWS::IAM::ManagedPolicy
    Properties: 
      Description: Allow access to all S3 buckets, except catpics
      ManagedPolicyName: AllowAllS3ExceptCats
      PolicyDocument: 
        Version: 2012-10-17
        Statement:
          - Effect: Allow
            Action: 's3:*'
            Resource: '*' 
          - Effect: Deny
            Action: 's3:*'
            Resource: [ !GetAtt catpics.Arn, !Join ['', [!GetAtt catpics.Arn, '/*']]]
Outputs:
  catpicsbucketname:
    Description: Bucketname for catpictures (the best animal!)
    Value: !Ref catpics
  animalpicsbucketname:
    Description: Bucketname for animalpics (the almost best animals!)
    Value: !Ref animalpics
  sallyusername:
    Description: IAM Username for Sally
    Value: !Ref sally
```

Stack Name: IAM \
Parameter: sallypassword (choose a PW) \
Next

Next

Because this Stack will create an IAM User, scroll all the way down and acknowledge this "Capability" \
Create Stack

/
Click on the Resources tab and see the (4) resources that have been created. \
S3 Buckets: animalpics & catpics \
IAM Managed Policy: policy \
IAM User: sally

Right Click on Sally's Physical ID & open in new tab \
Notice the attched policy IAMUserChangePassword

**Notice the effects of the Managed Policy:** \
CloudFormation → Stacks → click on the IAM stack → Outputs tab → copy the Value for the Sally User Name \
Open a New Browser, or Private Tab \
Log into AWS console using the Sally User Name Value \
Now you'll be prompted to change the PW \
Notice that you don't have any permissions to access EC2, or S3, or any other permissions!

**Assign S3 Full Admin as an Inline Policy:** \
Copy the following into your clipboard: \
```
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
} 
```

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

\
Go back to where you're logged into your iamadmin \
IAM → Users → Permissions tab → Delete the Inline Policy

\
Go back to where you're logged in as Sally \
S3 → Buckets & Refresh \
Notice the Access Deny error (as expected)

\
**Attaching a Managed Policy:** /
Go back to where you're logged into your iamadmin \
IAM → Users → Permissions tab → Add Permissions (to add a Managed Policy) → Attach Existing Policies Directly \
Search: AllowAllS3ExceptCats (select this) \
Next: Review \
Add Permission

Click on the AllowAllS3ExceptCats policy → {} JSON \
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": "s3:*",
            "Resource": "*",
            "Effect": "Allow"
        },
        {
            "Action": "s3:*",
            "Resource": [
                "arn:aws:s3:::iam-catpics-z157hoyizcsb",
                "arn:aws:s3:::iam-catpics-z157hoyizcsb/*"
            ],
            "Effect": "Deny"
        }
    ]
} 
```

Go back to where you're logged in as Sally \
S3 → Buckets & Refresh \
Attempting to Access the catpics Bucket will be Denied, \
but you do still have Access to the animalpics Bucket! You can add/view any files

\
**Clean Up!!** \
Go back to where you're logged into your iamadmin \
IAM → Users → Sally → Delete Sally's Managed Policy \
S3 → Buckets → select the catpics Bucket → Empty \
S3 → Buckets → select the animalpics Bucket → Empty \
CloudFormation → Stacks → select the Stack → Delete → Delete Stack

**Make sure there are no errors for the Stack Deletion!!**
Click on the Stack → Events tab \
Make sure there are no errors in the Status field \
If we forgot to delete the files in the Buckets before deleting the Stack, there would likely be an error. \
Also, if we didn't delete the Managed Policy, we wouldn't have been able to delete Sally!
