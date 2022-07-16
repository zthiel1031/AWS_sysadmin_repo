# Emailing using AWS Lambda for S3 bucket uploads

This will serve as a walkthrough for using AWS Lambda in conjunction with SES to email an employee everytime a file is uploaded to a specific S3 bucket. 

As you progress through the walkthrough please note that any ![#f01515](red highlighted text) `#f01515` will be the clickable options in the management console and any ![#deaa10]yellow highlighted text) `#deaa10` will be the fill in variables.


## Steps to deploying lambda function

- set IAM policy and execution role
- verify email identity
- create lambda function for sending email through SES 
- create S3 bucket
- add trigger


# IAM policy and Execution role

In order to access and use the SES service we must first create a ![#f01515](Role) `#f01515` and a ![#f01515](Policy) `#f01515`.

To create a `policy` follow these steps:


1. Navigate to ![#f01515](IAM) `#f01515` in the management console.

![](Images/IAM.JPG)

2. On the left-hand side, click ![#f01515](Policies) `#f01515` then ![#f01515](Create policy) `#f01515`

![](Images/policies.JPG)

3. After clicking on create policy, select the ![#f01515](JSON) `#f01515` tab. Delete everything in the box and enter this code:

```json
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

4. Next click on ![#f01515](Tags) `#f01515`
    You can elect to continue without tagging however, it is not recommended, as tags help keep things organized.

![](Images/JSON.JPG)

5. After applying the appropriate tags for the `Key` we will use ![#deaa10]email_project) `#deaa10` and the `Value` we will use ![#deaa10]ses_lambda_allow) `#deaa10`.

6. Click on ![#f01515](Review) `#f01515` to continue to the final step in the policy creation.

![](Images/tags.JPG)

7. Finally we can give our `policy` a `name` in this case we will use ![#deaa10]Lambda_SES) `#deaa10` and a meaningful `description` then click on ![#f01515](Create policy) `#f01515`.

![](Images/policy_create.JPG)

---

Next we will create a `Role` to give `Lambda` permission to access the `SES` service through our `policy`.

To create a `Role` follow these steps:

1. Navigate to ![#f01515](IAM) `#f01515` in the management console.

![](Images/IAM.JPG)

2. On the left-hand side, click ![#f01515](Roles) `#f01515` then ![#f01515](Create role) `#f01515`

![](Images/roles.JPG)

3. Since we are using `Lambda` with `SES` we can set `Trusted entity type` as ![#deaa10]AWS service) `#deaa10` if not already selected. We can set our use case as ![#deaa10]Lamda) `#deaa10` and click ![#f01515](Next) `#f01515`

![](Images/role_entity.JPG)

4. Next select our policy that was created in the `Policy` section. Under `Permissions policies` select our ![#deaa10]Lamda_SES) `#deaa10` policy and click ![#f01515](Next) `#f01515`.

![](Images/role_permission.JPG)

5. Now give a name to `Role`. In this exmaple the name ![#deaa10]lambda_allow_ses) `#deaa10` will be used. Then fill in tags `Key` is ![#deaa10]email_project) `#deaa10` and `Value` is ![#deaa10]lambda_iam) `#deaa10`. Click ![#f01515](Create role) `#f01515` after finishing.

![](Images/role_name.JPG)

![](Images/role_tag_create.JPG)


# Verify Email Identity

In order to use the `SES` service, the email sender and recipient address identities must be created and verified.

To do so follow these steps:

1. Navigate to ![#f01515](Amazon Simple Email Service) `#f01515` in the management console.

![](Images/IAM_SES.JPG)

2. In the left-hand pane click ![#f01515](Verified Identities) `#f01515` then click ![#f01515](Create identity) `#f01515`

![](Images/ses_create_identity.JPG)

3. For `Identity type` select ![#deaa10]Email address) `#deaa10`. In the box below, enter the email address to send or recieve the email, ![#deaa10]aws.zthiel1031@gmail.com) `#deaa10` will be used for this example.

4. Tags again are optional, but for consistency and organization sake, tag the `Key` as ![#deaa10]email_project) `#deaa10` and the `Value` as ![#deaa10]email-1) `#deaa10`. Then click ![#f01515](Create identity) `#f01515`.

![](Images/create_identity.JPG)

## Repeat steps 2-4 for a second identity
### In our case ![#deaa10]aws.test.zthiel1031@gmail) `#deaa10` was used as the second identity. 

5. After creating the `Identities` navigate to your email inbox and look for an email from `Amazon Web Services` inside click on the ![#f01515](hyperlink) `#f01515` to authorize this email to be used with `SES`

![](Images/email_hyperlink.JPG)

After doing this screen will be shown to confirm email authorization.

![](Images/email_verify.JPG)


# Create Lambda function and test

Now lets move on to create our `Lambda` function.

To do so follow these steps:

1. Navigate to ![#f01515](Lambda) `#f01515` in the management console.

![](Images/Lambda.JPG)

2. On the left-hand pane click on ![#f01515](Functions) `#f01515` then click ![#f01515](Create function) `#f01515`.

![](Images/function.JPG)

3. Select ![#deaa10]Author from scratch) `#deaa10`, then enter a function name. ![#deaa10]email_ses) `#deaa10` will be used in this example. Leave `Runtime` as ![#deaa10]Node.js 16.x) `#deaa10` and select ![#deaa10]x86_64) `#deaa10` for `Architecture`.
Under `Advanced settings` check the box ![#f01515](Enable Tags) `#f01515` and create tags for this function. ![#deaa10]email_project) `#deaa10` will be the `Key` and ![#deaa10]send_ses_lambda) `#deaa10` will be the `Value`. After all that is done click ![#f01515](Create function) `#f01515`.

![](Images/author_name_arch.JPG)

![](Images/tags_create.JPG)

4. On the left-hand pane click on ![#f01515](Functions) `#f01515` then click the newly created function ![#f01515](email_ses) `#f01515`.

![](Images/email_ses.JPG)

5. Scroll down and click on ![#f01515](Configuration) `#f01515` then, ![#f01515](Permissions) `#f01515`. Make sure the `Role name` matches the role we created above. If it does not, click on ![#f01515](Edit) `#f01515` and under `Existsing role` at the bottom, select the role from the drop-down.

![](Images/config_permissions.JPG)

![](Images/edit_permissions.JPG)

6. Click on ![#f01515](Code) `#f01515`. Replace the pre-existing code with this:

```json
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

7. After the code is entered click ![#f01515](Deploy) `#f01515`, then ![#f01515](Test) `#f01515`. A `Configure test event` window will pop up. Set `Test event action` to ![#deaa10]Create new event) `#deaa10` and give the event a name. ![#deaa10]TestEvent) `#deaa10` will be used for the example. Leave everything as it is and click ![#f01515](Save) `#f01515`

![](Images/test_event.JPG)

8. Click ![#f01515](Test) `#f01515`. After about 30 seconds the results will show. Look for green `Succeeded` or red `Failure`.

![](Images/success_lambda.JPG)


# Create S3 Bucket

To create an S3 Bucket follow these steps:

1. Navigate to ![#f01515](S3) `#f01515` in the management console.

![](Images/S3.JPG)

2. On the left-hand pane click on ![#f01515](Buckets) `#f01515` then click ![#f01515](Create bucket) `#f01515`.

![](Images/bucket_create.JPG)

3. Give the bucket a unique name. ![#deaa10]testbucketdemo123456789) `#deaa10` will be used in this example

![](Images/bucket_name.JPG)

4. Make sure to `Block alll public access`, `Enable Bucket Versioning`, create `tags` and click![#f01515](Create bucket) `#f01515`.
 

![](Images/block_tag_version.JPG)
 
# Add trigger and test

1. Navigate to ![#f01515](Lambda) `#f01515` in the management console.

![](Images/Lambda.JPG)

2. On the left-hand pane click on ![#f01515](Functions) `#f01515` then click the newly created function ![#f01515](email_ses) `#f01515`.

![](Images/email_ses.JPG)

3. Click on ![#f01515](+ Add trigger) `#f01515`.

![](Images/trigger_add.JPG)

4. Select `S3` as trigger type. Fill in the bucket name that was created. ![#deaa10]testbucketdemo123456789) `#deaa10` was used in this example. And change `Event type` to ![#f01515](PUT) `#f01515`. This will ensure that every time a new file is added, the `Lambda` function gets triggered. Then click ![#f01515](Add) `#f01515`.

![](Images/trigger_type.JPG)

Here is the new function overview:

![](Images/trigger_map.JPG)

Now to test the function

5. Navigate to ![#f01515](S3) `#f01515` in the management console.

![](Images/S3.JPG)

6. On the left-hand pane click on ![#f01515](Buckets) `#f01515` then click ![#f01515](testdemobucket123456789) `#deaa10`.

![](Images/bucket_select.JPG)

7. Click ![#f01515](Upload) `#f01515`.

![](Images/file_upload.JPG)

8. Click ![#f01515](Add files) `#f01515` and select a file to upload.

![](Images/add_file.JPG)

9. Once the green status bar is shown. check the email listed under `ToAddresses`.

![](Images/upload_success.JPG)

![](Images/email_receive.JPG)