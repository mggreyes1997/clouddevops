
# Hands-on S3-02 : S3 Lifecycle Management and Bucket Replication

Purpose of the this hands-on training is to review how to to create a S3 bucket, manage lifecycle of objects and to implement bucket replication.

## Learning Outcomes

At the end of the this hands-on training, students will be able to;

- create a S3 bucket.

- manage lifecycle of objects in S3.

- manage IAM role for bucket replication.

- set the S3 bucket replica.

## Outline

- Part 1 - Creating Bucket for Lifecycle Management and setting Lifecycle Management

- Part 2 - Creating Source and Destination S3 Bucket

- Part 3 - Creating IAM Role for Bucket Replication

- Part 4 - Configuring (Entire) Bucket Replica

- Part 5 - Configuring (Tags, Prefix) Bucket Replica


## Part 1 - Creating S3 Bucket for Lifecycle Management and setting Lifecycle Management


- Create a new bucket named `YOURNAME-lifecycle` with following properties.

```text
Block all public access   : CHECKED
Versioning                : Disabled
Tagging                   : 0 Tags
Default encryption        : Keep default
Object lock               : Disabled
```

- Click the S3 bucket `YOURNAME-lifecycle` and upload following files.

```text
index.html
cat.jpg
```

- Create new folders as "newfolder"

- Open lifecycle rules from the management tab and create lifecycle rule with following configuration.
```
MANAGEMENT>>>LIFECYCLE>>>>>CREATE LIFECYCLE RULE
```
Lifecycle rule configuration:
```text
Lifecycle rule name: LCRule
Choose a rule scope: Apply to all objects in the bucket
Check the box "I acknowledge that this lifecycle rule will apply to all objects in the bucket."
(Show that you can also select prefix as "newfolder" but skip it for now)
```
Lifecycle rule actions:
```text
Select "Transition current versions of objects between storage classes" and "Expire current versions of objects"
```
Transition current versions of objects between storage classes
```text
Choose storage class transitions: "One Zone IA" ---> Days after object creation "30" days
```
Expire current versions of objects:
```text
Days after object creation: 365
```
Hit "Create rule" button.



## Part 2 - Creating Source and Destination S3 Bucket

### Step 1 - Create Source Bucket

- Open S3 Service from AWS Management Console.

- Create a bucket of `YOURNAME-source` with following properties,

```text
Region                      : us east-1 (N.Virginia)
Block all public access     : CHECKED 
Versioning                  : Enable (Replication requires versioning to be enabled for the source and destination bucket.)
Tagging                     : 0 Tags
Default encryption          : Keep default
Object lock                 : Disabled

PS: Please, do not forget to select "US East (N.Virginia)" as Region
```


### Step 2 - Create Destination Bucket

- Create a new bucket named `YOURNAME-destination` with following properties.

```text
Region                    : us-east-2 (Ohio)
Block all public access   : CHECKED
Versioning                : Enable (Replication requires versioning to be enabled for the source and destination bucket.)
Tagging                   : 0 Tags
Default encryption        : Keep default
Object lock               : Disabled


PS: Please, do not forget to select "US East (Ohio)" as Region
```


## Part 3 - Creating IAM Role for Bucket Replication

- Go to `IAM Service` on AWS management console.

- Click `Policy` on left-hand menu and select `create policy`.

- Select `JSON` option and paste the policy seen below.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "s3:Get*", 
                "s3:ListBucket"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s3:::YOURNAME-source(change me)",
                "arn:aws:s3:::YOURNAME-source(change me)/*"
            ]
        },
        {
            "Action": [ 
                "s3:ReplicateObject", 
                "s3:ReplicateDelete",
                "s3:ReplicateTags",
                "s3:GetObjectVersionTagging"
            ],
            "Effect": "Allow",
            "Resource": "arn:aws:s3:::YOURNAME-destination(change me)/*"
        }
    ]
}
```

- Enter the followings as policy name and description.

```text
Name            : crr-policy

Description     : crr-policy
```

- Click `create policy`.

- Go to `Roles` on the left hand menu and click `create role`.

```text
Type of Trusted Entity      : AWS Service
Use Case                    : S3
```

- Click next.

- Enter `crr-policy` in filter policies box and select the policy

- Click Next `Tag`

```text
Keep it as default
```

- Click Next:Review

- Enter the followings as role name and description an review

```text
Role Name           : crr-role
Role Description    : crr-role
```

- Click `create role`.

## Part 4 - Configuring (Entire) Bucket Replica

### Step 1 - Configuring Bucket

- Go to S3 bucket on AWS Console.

- Select `YOURNAME-source`.

- Select `Management` >>`Replication Rules` >> `Create Replication Rule`

Replication rule configuration:
```text
Replication Rule Name: FirstReplicationRule
Status               : Enable
Priority             : 0
```
Source bucket:
```text
Choose a rule scope : Apply to all objects in the bucket
```
Destination:
```text
Choose a bucket in this account
Bucket name : YOURNAME-destination
```
IAM role:
```text
Choose from existing IAM roles : crr-role
```
Leave other settings as is.


### Step 2 - Testing (Entire Bucket)

- Go to S3 bucket `YOURNAME-source` and upload following files.

```text
index.html
cat.jpg
```

- Go to `YOURNAME-destination` bucket, show the objects are replicated from source bucket.

## Part 5 - Configuring (Tag and Prefix) Bucket Replica

### Step 1 - Configuring with Prefix

- Go to S3 bucket on AWS Console

- Select `YOURNAME-source`.

- Create a folder named `kitten`.

- Select `Management` >> `Replication rules` >> `Create replication rule`

Replication rule configuration:
```text
Replication Rule Name: PrefixRule
Status               : Enable
Priority             : 1
```
Source bucket:
```text
Choose a rule scope : Limit the scope of this rule using one or more filters
Prefix              : kitten
```
Destination:
```text
Choose a bucket in this account
Bucket name : YOURNAME-destination
```
IAM role:
```text
Choose from existing IAM roles : crr-role
```
Leave other settings as is.

Click save.

List replication rules and disable the `FirstReplicationRule`.

### Step 2 - Testing (Prefix)

- Go to `YOURNAME-source` bucket and upload `put-out-of-kitten.txt` under the bucket.

- Go to `YOURNAME-destination` bucket and show that the`put-out-of-kitten.txt` is not replicated in the bucket.

- This time go to `YOURNAME-source` bucket, select `kitten` folder and upload `put-into-kitten.txt` under kitten folder.

- Go to `YOURNAME-destination` bucket show that `kitten` folder has been replicated automatically due to replication rule.

### Step 3 - Configuring with tag

- Go to S3 bucket on AWS Console

- Select `YOURNAME-source`.

- Select `Management` >> `Replication rules` >> `Create replication rule`

Replication rule configuration:
```text
Replication Rule Name: TagRule
Status               : Enable
Priority             : 2
```
Source bucket:
```text
Choose a rule scope : Limit the scope of this rule using one or more filters
Tags                : Add tag (Key: Name, Value: cat)
```
Destination:
```text
Choose a bucket in this account
Bucket name : YOURNAME-destination
```
IAM role:
```text
Choose from existing IAM roles : crr-role
```
Leave other settings as is.

Click save.

Show that there are three rules and the first rule is disabled. Disable the `PrefixRule`.

### Step 2 - Testing (Tag)

- Go to `YOURNAME-source` bucket and direct upload `no-tag-file.txt`.

- Go to `YOURNAME-destination` bucket and show that the `no-tag-file.txt` is not replicated in the bucket.

- This time go to `YOURNAME-source` bucket, and upload the `tag-file.txt` file with tag as `Key:Name` and `Value:cat` under Properties.

- Go to `YOURNAME-destination` bucket show that the `tag-file.txt` file is replicated.
