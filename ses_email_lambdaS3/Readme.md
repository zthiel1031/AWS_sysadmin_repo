# Emailing using AWS Lambda for S3 bucket uploads

This will serve as a walkthrough for using AWS Lambda in conjunction with SES to email an employee everytime a file is uploaded to a specific S3 bucket. 

## Steps to deploying lambda function

- set IAM policy and execution role
- verify email identity
- create lambda function for sending email through SES 
- create S3 bucket
- add trigger


# IAM policy and Execution role

In order to access and use the SES service we must first create a `Role` and a `Policy`.

To create a `policy` follow these steps:


1. Navigate to `IAM` in the management console.

![](Images/IAM.JPG)

2. On the left-hand side, click `Policies` then `Create policy`

![](Images/policies.JPG)

3. After clicking on create policy, select the `JSON` tab. Delete everything in the box and enter this code:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ses:SendEmail",
                "ses:SendRawEmail"
            ],
            "Resource": "*"
        }
    ]
}
```

4. Next click on `Tags`
    You can elect to continue without tagging however, it is not recommended, as tags help keep things organized.

![](Images/JSON.JPG)

5. After applying the appropriate tags for the `Key` we will use `email_project` and the `Value` we will use `ses_lambda_allow`.

6. Click on `Review` to continue to the final step in the policy creation.

![](Images/tags.JPG)

7. Finally we can give our `policy` a `name` in this case we will use `Lambda_SES` and a meaningful `description` then click on `Create policy`.

![](Images/policy_create.JPG)

---

Next we will create a `Role` to give `Lambda` permission to access the `SES` service through our `policy`.

To create a `Role` follow these steps:

1. Navigate to `IAM` in the management console.

![](Images/IAM.JPG)

2. On the left-hand side, click `Roles` then `Create role`

![](Images/roles.JPG)

3. Since we are using `Lambda` with `SES` we can set `Trusted entity type` as `AWS service` if not already selected. We can set our use case as `Lamda` and click `Next`

![](Images/role_entity.JPG)

4. Next select our policy that was created in the `Policy` section. Under `Permissions policies` select our `Lamda_SES` policy and click `Next`.

![](Images/role_permission.JPG)

5. Now give a name to `Role`. In this exmaple the name `lambda_allow_ses` will be used. Then fill in tags `Key` is `email_project` and `Value` is `lambda_iam`. Click `Create role` after finishing.

![](Images/role_name.JPG)

![](Images/role_tag_create.JPG)


# Verify Email Identity

In order to use the `SES` service, the email sender and recipient address identities must be created and verified.

To do so follow these steps:

1. Navigate to `Amazon Simple Email Service` in the management console.

![](Images/IAM_SES.JPG)

2. In the left-hand pane click `Verified Identities` then click `Create identity`

![](Images/ses_create_identity.JPG)

3. For `Identity type` select `Email address`. In the box below, enter the email address to send or recieve the email, `aws.zthiel1031@gmail.com` will be used for this example.

4. Tags again are optional, but for consistency and organization sake, tag the `Key` as `email_project` and the `Value` as `email-1`. Then click `Create identity`.

![](Images/create_identity.JPG)

## Repeat steps 2-4 for a second identity
### In our case `aws.test.zthiel1031@gmail` was used as the second identity. 

5. After creating the `Identities` navigate to your email inbox and look for an email from `Amazon Web Services` inside click on the `hyperlink` to authorize this email to be used with `SES`

![](Images/email_hyperlink.JPG)

After doing this screen will be shown to confirm email authorization.

![](Images/email_verify.JPG)


# Create Lambda function and test

Now lets move on to create our `Lambda` function.

To do so follow these steps:

1. Navigate to `Lambda` in the management console.

![](Images/Lambda.JPG)

2. On the left-hand pane click on `Functions` then click `Create function`.

![](Images/function.JPG)

3. Select `Author from scratch`, then enter a function name. `email_ses` will be used in this example. Leave `Runtime` as `Node.js 16.x` and select `x86_64` for `Architecture`.
Under `Advanced settings` check the box `Enable Tags` and create tags for this function. `email_project` will be the `Key` and `send_ses_lambda` will be the `Value`. After all that is done click `Create function`.

![](Images/author_name_arch.JPG)

![](Images/tags_create.JPG)

4. On the left-hand pane click on `Functions` then click the newly created function `email_ses`.

![](Images/email_ses.JPG)

5. Scroll down and click on `Configuration` then, `Permissions`. Make sure the `Role name` matches the role we created above. If it does not, click on `Edit` and under `Existsing role` at the bottom, select the role from the drop-down.

![](Images/config_permissions.JPG)

![](Images/edit_permissions.JPG)

6. Click on `Code`. Replace the pre-existing code with this:

```
// Copyright 2019 Amazon.com, Inc. or its affiliates. All Rights Reserved.
// SPDX-License-Identifier: Apache-2.0

var aws = require("aws-sdk");
var ses = new aws.SES({ region: "us-east-1" });
exports.handler = async function (event) {
  var params = {
    Destination: {
      ToAddresses: ["aws.test.zthiel1031@gmail.com"],
    },
    Message: {
      Body: {
        Text: { Data: "A new file has been added to the project-email-zthiel1031 bucket." },
      },

      Subject: { Data: "New File Upload" },
    },
    Source: "aws.zthiel1031@gmail.com",
  };
 
  return ses.sendEmail(params).promise()
};
```

# Please note!
### the `ToAddresses` and `Source` need to be changed to one of your verified identities.

![](Images/code_json.JPG)

7. After the code is entered click `Deploy`, then `Test`. A `Configure test event` window will pop up. Set `Test event action` to `Create new event` and give the event a name. `TestEvent` will be used for the example. Leave everything as it is and click `Save`

![](Images/test_event.JPG)

8. Click `Test`. After about 30 seconds the results will show. Look for green `Succeeded` or red `Failure`.

![](Images/success_lambda.JPG)


# Create S3 Bucket

To create an S3 Bucket follow these steps:

1. Navigate to `S3` in the management console.

![](Images/S3.JPG)

2. On the left-hand pane click on `Buckets` then click `Create bucket`.

![](Images/bucket_create.JPG)

3. Give the bucket a unique name. `testbucketdemo123456789` will be used in this example

![](Images/bucket_name.JPG)

4. Make sure to `Block alll public access`, `Enable Bucket Versioning`, create `tags` and click`Create bucket`.
 

![](Images/block_tag_version.JPG)
 
# Add trigger and test

1. Navigate to `Lambda` in the management console.

![](Images/Lambda.JPG)

2. On the left-hand pane click on `Functions` then click the newly created function `email_ses`.

![](Images/email_ses.JPG)

3. Click on `+ Add trigger`.

![](Images/trigger_add.JPG)

4. Select `S3` as trigger type. Fill in the bucket name that was created. `testbucketdemo123456789` was used in this example. And change `Event type` to `PUT`. This will ensure that every time a new file is added, the `Lambda` function gets triggered. Then click `Add`.

![](Images/trigger_type.JPG)

Here is the new function overview:

![](Images/trigger_map.JPG)

Now to test the function

5. Navigate to `S3` in the management console.

![](Images/S3.JPG)

6. On the left-hand pane click on `Buckets` then click `testdemobucket123456789`.

![](Images/bucket_select.JPG)

7. Click `Upload`.

![](Images/file_upload.JPG)

8. Click `Add files` and select a file to upload.

![](Images/add_file.JPG)

9. Once the green status bar is shown. check the email listed under `ToAddresses`.

![](Images/upload_success.JPG)

![](Images/email_receive.JPG)