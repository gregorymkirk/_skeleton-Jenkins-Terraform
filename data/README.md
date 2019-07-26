### Variable Files

Variable files must go in ./data to work withthe Jenkins Pipeline

bariable files must be named in the follwoing format:

*account-alias*-*aws_region*-*env*.tfvars

e.g.: **tfsawsdne02-us-east-1-poc.tfvars**

Valid Enviroment names are: 

* prod
* subprod
* stage
* test
* dev
* poc	

##Adding new supported environment names
If you wish to add different enviroments, then edit line 19 of the Jenkinsfile in the main directory

19         choice ( name: 'ENVIRONMENT', choices:['prod', 'subprod', 'stage', 'test', 'dev', 'poc'], description: 'Environment to build')
