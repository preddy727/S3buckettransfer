# Scenario

S3 bucket in one AWS account requires a transfer to another AWS account.  This is common during acquisitions and organizational changes. 

Objects encrypted using SSE-c encryption and the bucket owner does not have the keys that end user used to encrypt the object. 

### Steps to create source bucket with an SSE-c object

#Create your own bucket

Create a new bucket. The bucket name must be unique across all of Amazon S3. Therefore, you could use the following naming convention bucket-encrypt-ssec-<name>-<date>.

```bash
aws s3 mb s3://bucket-encrypt-ssec-<name>-<date>
```


#Create a local object

Create a sample object. This object is created unencrypted on your local environment (it will be encrypted by S3 during the upload operation).

```bash
echo "Test SSE-C" > ssec.txt
```


#Create your own key

Create a random key for encrypting the object.

```bash
openssl rand 32 -out ssec.key
```


#Put the object to S3

Using the following command, the object ssec.txt is uploaded to S3. S3 takes care of encrypting the object using the key ssec.key that you provide.

```bash
aws s3 cp ssec.txt s3://bucket-encrypt-ssec-<name>-<date>/ssec.txt --sse-c AES256 --sse-c-key fileb://ssec.key
```


Get the object from S3

Using the following command, the object ssec.txt is downloaded from S3. S3 takes care of decrypting the object using the key ssec.key that you provide.

```bash
aws s3 cp s3://bucket-encrypt-ssec-<name>-<date>/ssec.txt ssec-downloaded.txt --sse-c AES256 --sse-c-key fileb://ssec.key
 ```
 
Notice that you cannot download the object without providing the same key that you provided for the upload operation. Therefore, it is pivotal that the you manage your keys in a highly durable and highly available way.
You can check the content of the object downloaded from S3 by running the following command.

```bash
cat ssec-downloaded.txt
```


*Enable bucket versioning.* 

```bash
aws s3api put-bucket-versioning —bucket bucket-encrypt-ssec-q2-3823 —versioning-configuration Status=Enabled
```
