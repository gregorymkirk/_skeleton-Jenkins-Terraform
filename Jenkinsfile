pipeline {

    agent any

    environment {
        TERRAFORM_VER = 'terraform_0.11.14'
        TERRAFORM_HOME = "~/.local/bin/${env.TERRAFORM_VER}"
        //this should be the name of the Git repo
        SOLUTION = 'CHANGEME'


    }

    parameters {
        //cusotmize this list with the acceptable AWS account aliases
        choice ( name: 'ACCOUNTALIAS', choices: ['awsaccount1', 'awsaccount2', 'awsaccount3'], description: 'Account Alias')
        choice ( name: 'AWS_REGION', choices:['us-east-1', 'us-west-2'], description: 'AWS Region')
        //customise the list with the client's enviroment choices
        choice ( name: 'ENVIRONMENT', choices:['prod', 'subprod', 'stage', 'test', 'dev' ], description: 'Environment to build')
    }



    stages {

        stage('validate') {
            steps{
                script{
                    //make sure we have customised this for the repo
                    if (env.SOLUTION == 'CHANGEME') {
                        currentBuild.result = 'FAILED'
                        return
                    }
                }    
            }
        }

        stage('Checkout SCM Code') {
            steps {
                checkout scm
            }
        }

        stage('terraform init') {
            steps {
                //this will need to be changed when you move to artifactory, replace the aws s3 cp with curl (see artifactory)
                // the terraform.zip file needs to be named: terraformXX.zip, e.g. terraform11.zip so that env.TERRAFORM_VER works correctly
                // sh "[ -f ${env.TERRAFORM_HOME}/terraform ] && echo 'terraform is installed... Moving forward' || ~/.local/bin/aws s3 cp s3://tfs-terraform-state/binary/${env.TERRAFORM_VER}.zip ${env.TERRAFORM_VER}.zip && unzip -o ${env.TERRAFORM_VER}.zip && chmod +x ./terraform && mkdir ${env.TERRAFORM_HOME} && cp terraform ${env.TERRAFORM_HOME}/ "
                sh """
                    if [ -f ${env.TERRAFORM_HOME}/terraform ]; then
                    echo 'terraform is installed... Moving forward'
                    else
                    ~/.local/bin/aws s3 cp s3://tfs-terraform-state/binary/${env.TERRAFORM_VER}.zip ${env.TERRAFORM_VER}.zip
                    unzip -o ${env.TERRAFORM_VER}.zip && chmod +x ./terraform
                    mkdir ${env.TERRAFORM_HOME}
                    cp terraform ${env.TERRAFORM_HOME}/
                    fi
                    """
                sh "${env.TERRAFORM_HOME}/terraform init -upgrade -backend-config=\"bucket=tf-state-${params.ACCOUNTALIAS}\" -backend-config=\"key=states/${env.SOLUTION}/${params.AWS_REGION}-${params.ENVIRONMENT}.state\""
            }
        }
        stage('terraform plan') {
            steps {
                sh "${env.TERRAFORM_HOME}/terraform plan -var-file=\"./data/${params.ACCOUNTALIAS}-${params.AWS_REGION}-${params.ENVIRONMENT}.tfvars\" -var=\"aws_region=${params.AWS_REGION}\" "

                input (message: 'Ready to apply?', ok: 'Yes')
            }
        }

        stage('terraform apply') {
            steps {
                sh "${env.TERRAFORM_HOME}/terraform apply -auto-approve -var-file=\"./data/${params.ACCOUNTALIAS}-${params.AWS_REGION}-${params.ENVIRONMENT}.tfvars\" -var=\"aws_region=${params.AWS_REGION}\""
            }
        }        
    }

}
