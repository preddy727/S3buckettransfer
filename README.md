# Scenario

S3 bucket in one AWS account requires a transfer to another AWS account.  This is common during acquisitions and organizational changes. 

Objects encrypted using SSE-c encryption and the bucket owner does not have the keys that end user used to encrypt the object. 

### Pre-req - Steps to create source bucket with an SSE-c object to setup Scenario 

## Create your own bucket

Create a new bucket. The bucket name must be unique across all of Amazon S3. Therefore, you could use the following naming convention bucket-encrypt-ssec-<name>-<date>.

```bash
aws s3 mb s3://bucket-encrypt-ssec-<name>-<date>
```


## Create a local object

Create a sample object. This object is created unencrypted on your local environment (it will be encrypted by S3 during the upload operation).

```bash
echo "Test SSE-C" > ssec.txt
```


## Create your own key

Create a random key for encrypting the object.

```bash
openssl rand 32 -out ssec.key
```


## Put the object to S3

Using the following command, the object ssec.txt is uploaded to S3. S3 takes care of encrypting the object using the key ssec.key that you provide.

```bash
aws s3 cp ssec.txt s3://bucket-encrypt-ssec-<name>-<date>/ssec.txt --sse-c AES256 --sse-c-key fileb://ssec.key
```


## Get the object from S3

Using the following command, the object ssec.txt is downloaded from S3. S3 takes care of decrypting the object using the key ssec.key that you provide.

```bash
aws s3 cp s3://bucket-encrypt-ssec-<name>-<date>/ssec.txt ssec-downloaded.txt --sse-c AES256 --sse-c-key fileb://ssec.key
 ```
 
Notice that you cannot download the object without providing the same key that you provided for the upload operation. Therefore, it is pivotal that the you manage your keys in a highly durable and highly available way.

 ## You can check the content of the object downloaded from S3 by running the following command.

```bash
cat ssec-downloaded.txt
```


## Enable bucket versioning

```bash
aws s3api put-bucket-versioning —bucket bucket-encrypt-ssec —versioning-configuration Status=Enabled
```

# Setup an intermediary bucket in separate AWS account
 
## Enable bucket versioning enabled
 
 
## Enable receiving replicated objects from a source bucket

Sign in to the AWS Management Console and open the Amazon S3 console at https://console.aws.amazon.com/s3/

* In the left navigation pane, choose *Buckets*.
* In the *Buckets* list, choose the bucket that you want to use as a destination bucket.
* Choose the *Management* tab, and scroll down to *Replication rules*.
* For *Actions*, choose *Receive replicated objects*. 
    Follow the prompts and enter the AWS account ID of the source bucket account and choose *Generate policies*. This will generate an Amazon S3 bucket policy and a KMS key policy.
* To add this policy to your existing bucket policy, either choose *Apply settings* or choose *Copy* to manually copy the changes.
 
 ## Create an IAM role with an IAM Policy for replication. Replace the source and intermediary bucket below.  
 
 # IAM Policy example below
 ```json
 {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "s3:ListBucket",
                "s3:GetReplicationConfiguration",
                "s3:GetObjectVersionForReplication",
                "s3:GetObjectVersionAcl",
                "s3:GetObjectVersionTagging",
                "s3:GetObjectVersion",
                "s3:ObjectOwnerOverrideToBucketOwner"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s3:::bucket-encrypt-ssec",
                "arn:aws:s3:::bucket-encrypt-ssec*"
            ]
        },
        {
            "Action": [
                "s3:ReplicateObject",
                "s3:ReplicateDelete",
                "s3:ReplicateTags",
                "s3:GetObjectVersionTagging",
                "s3:ObjectOwnerOverrideToBucketOwner"
            ],
            "Effect": "Allow",
            "Resource": "arn:aws:s3:::intermediarybucket2/*"
        }
    ]
}
```
 ## Setup a replication rule on the Source bucket 
 
 
 ## Setup a Batch replication policy using the sample below. Replace with source bucket, manifest destination bucket and completion job destination bucket. 
 ```json
 
 
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "s3:InitiateReplication"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s3:::bucket-encrypt-ssec-q2-3823/*"
            ]
        },
        {
            "Action": [
                "s3:GetReplicationConfiguration",
                "s3:PutInventoryConfiguration",
                "s3:GetObjectVersionForReplication"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s3:::bucket-encrypt-ssec"
            ]
        },
        {
            "Action": [
                "s3:GetObject",
                "s3:GetObjectVersion"
            ],
            "Effect": "Allow",
            "Resource": [
                "arn:aws:s3:::app2container-pr-mar-30/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::app2container-pr-mar-30/*",
                "arn:aws:s3:::app2container-pr-mar-30/*"
            ]
        }
    ]
}

 ## Verify data in intermediary bucket and delete the Source bucket 
 
 ## Recreate the original source bucket name in the new Account 
 
 ## Setup replication rule and batch job from intermediary bucket to the new bucket (using original name) in new account
 
 ## Run the batch job and verify data 

