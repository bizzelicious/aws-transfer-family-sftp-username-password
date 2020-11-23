## Deploy options ##

### Option 1 - CLI: ###
```
FTP_STACKNAME=<stack-name, eg my-sftp-server>
AWS_PROFILE=<profilename, eg default>
AWS_REGION=<region, eg eu-west-1>
BUCKETNAME=temp-deploy-bucket-$RANDOM
aws --profile $AWS_PROFILE --region $AWS_REGION s3 mb s3://$BUCKETNAME &&
aws --profile $AWS_PROFILE --region $AWS_REGION cloudformation package \
        --template-file ftp.yml \
        --output-template-file template.ftp.yml \
        --s3-bucket $BUCKETNAME &&
aws --profile $AWS_PROFILE --region $AWS_REGION cloudformation deploy \
        --template-file template.ftp.yml \
        --stack-name $FTP_STACKNAME \
        --capabilities CAPABILITY_IAM &&
aws --profile $AWS_PROFILE --region $AWS_REGION s3 rb s3://$BUCKETNAME --force &&
rm -f template.ftp.yml
```

### Option 2 - Manual: ###
1. Optional: Create bucket for index.py 
2. Upload index.py to bucket
3. In ftp.yml, change the Lambda function code URI:
```
  GetUserConfigLambda:
    Type: AWS::Lambda::Function
    Properties:
      Code: index.py
      Description: A function to lookup and return user data from AWS Secrets Manager.
```
To:
```
  GetUserConfigLambda:
    Type: AWS::Lambda::Function
    Properties:
      Code:
        S3Bucket: <your-bucket-name>
        S3Key: index.py
      Description: A function to lookup and return user data from AWS Secrets Manager.
```
(https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-lambda-function-code.html)
4. Create a stack using the `ftp.yml` template.


### Create example user ###
1. Create a stack using the `example-user.yml` template, change the parameter value `<ftp-stack-name>` to the $FTP_STACKNAME you choosed for the ftp stack.
Using CLI in the same shell as the first deployment:
```
aws --profile $AWS_PROFILE --region $AWS_REGION cloudformation deploy \
        --template-file example-user.yml \
        --stack-name sftp-example-user \
        --capabilities CAPABILITY_IAM \
        --parameter-overrides UserPassword=changeme BucketParameter=$FTP_STACKNAME-Bucket SftpUserRoleParameter=$FTP_STACKNAME-SftpUserRole TransferServerParameter=$FTP_STACKNAME-TransferServer
```

## Read more ##
* https://aws.amazon.com/blogs/storage/enable-password-authentication-for-aws-transfer-family-using-aws-secrets-manager-updated/
* https://aws.amazon.com/blogs/storage/simplify-your-aws-sftp-structure-with-chroot-and-logical-directories/
