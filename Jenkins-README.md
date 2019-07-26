### Skeleton for TFS terraform repos

##Included file:
* .gitinore - uses recomended terrform settings
* .backend.tf  - must be included to allow jenkins to configure the jobs to the S3 shared state
* Jenkinsfile - This handles all the needed steps to manage the terraform pipeline.

##Usage notes:

The Jenkinsfile **must** be edited.
 Line 10:         SOLUTION = 'CHANGEME'
 Change CHAGNEME to the name of your project repo to ensure uniqueness.  If you not, it is possible you coudl overwrite the statefile of another project.
 *we are debugging code that will fail the build in this case*


Before running the pipeline you must create an S3 bucket for the storage of the shared state.  
This bucket must be named  tf-state-*account-alias*
e.g. : tf-state-tfsawsdne02

The  YAML cloudformation template below can be used to  create the  S3 bucket for the account.
* You may enter the ID of a previously created KMS key if you wish to use KMS manage encryption on this bucket.  I fyou do not enter a KMS key ID it will create the bucket using an AWS managed key.
* Note you will need to manually enter the account alias. (Cloudformation does not have a psuedo-paramenter for the alias)

##Variable Files

Variable files (\*.tfvars) must be stored in ./data

see  ./data/README.md for more information on naming


##Cloudformation Template (**(run only once per ACCOUNT**)
```
AWSTemplateFormatVersion: "2010-09-09"
Description: "Creates the S3 bucket for Terraform statefiles"

Parameters:
  AccountAlias:
    Type : "String"
    Description: "Type in the Account Alias, this will be used as part of the bucket name to ensure uniqueness"
  KMSKey:
    Type: "String"
    Description: " Optiinal - The ID of the KMS master key to use for bucket encryption, if none specified, will use AES encrytion"

Conditions:
  AES: 
    !Equals [ !Ref "KMSKey", "" ]
  KMS:
    !Not  [ !Equals [ !Ref "KMSKey", "" ]]

Resources: 
  TFSharedStateBucketKMS:
    Condition: KMS
    Type: AWS::S3::Bucket
    Properties: 
      AccessControl: Private
      BucketEncryption: 
        ServerSideEncryptionConfiguration: 
          - 
            ServerSideEncryptionByDefault: 
              KMSMasterKeyID: !Ref "KMSKey"
              SSEAlgorithm: "aws:kms"
      BucketName: !Join [ "-" , [ "tf-state", !Ref "AccountAlias" ] ]
      PublicAccessBlockConfiguration:
        BlockPublicAcls: True
        BlockPublicPolicy: True
        IgnorePublicAcls: True
        RestrictPublicBuckets: True
      VersioningConfiguration:
        Status: Enabled


  TFSharedStateBucketAES:
    Condition: AES
    Type: AWS::S3::Bucket
    Properties: 
      AccessControl: Private
      BucketEncryption: 
        ServerSideEncryptionConfiguration: 
          - 
            ServerSideEncryptionByDefault: 
              SSEAlgorithm: "AES256"
      BucketName: !Join [ "-" , [ "tf-state", !Ref "AccountAlias" ] ]
      PublicAccessBlockConfiguration:
        BlockPublicAcls: True
        BlockPublicPolicy: True
        IgnorePublicAcls: True
        RestrictPublicBuckets: True
      VersioningConfiguration:
        Status: Enabled

Outputs:
  Statebucket : 
    Condition: KMS
    Description: "The S3 bucket created for terraform state files"
    Value: !Ref "TFSharedStateBucketKMS"
    Export:
      Name: 'TerraformStateBucket'

  Statebucket : 
    Condition: AES
    Description: ":The S3 bucket created for terraform state files"
    Value: !Ref "TFSharedStateBucketAES"
    Export:
      Name: 'TerraformStateBucket'

```

