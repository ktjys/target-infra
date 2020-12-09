pipeline {
  agent {
    kubernetes {
      yaml '''
kind: Pod
metadata:
  name: aws-cli
  labels:
    app: awscli
spec:
  containers:
  - name: cli
    image: amazon/aws-cli:latest
    command:
    - /bin/sh
    tty: true
'''
    }

  }
  stages {
    stage('Prepare') {
      steps {
        sh '''
curl https://nexus.acldevsre.de/repository/target-infra/target-infra/target-infra-tar/v0.0.1/target-infra-tar-v0.0.1.tar -o terraform.tar 
tar xvf terraform.tar
'''
        sh '''
whoami
pwd
            KUBECTL=~/bin/kubectl
            if [ -f "$KUBECTL" ]; then
              echo "$KUBECTL exists"
            else
              mkdir ~/bin -p
              curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
              chmod +x ./kubectl
              mv ./kubectl $KUBECTL
export PATH=$PATH:/home/jenkins/bin
ls -al ~/bin
              kubectl version --short --client
            fi
          '''
      }
    }

    stage('Terraform plan') {
      steps {
        withVault(configuration: [vaultUrl: 'https://dodt-vault.acldevsre.de',  vaultCredentialId: 'approle-for-vault', engineVersion: 2], vaultSecrets: [[path: 'jenkins/tjk', secretValues: [[envVar: 'AWS_ACCESS_KEY_ID', vaultKey: 'aws_access_key_id'],[envVar: 'AWS_SECRET_ACCESS_KEY', vaultKey: 'aws_secret_access_key'],[envVar: 'AWS_SESSION_TOKEN', vaultKey: 'aws_session_token']]]]) {
          sh '''
cd terraform/Target_infra
terraform init
terraform plan
'''
        }

      }
    }

    stage('Deploy Approval') {
      steps {
        input(message: 'Are you Confirm?', submitter: 'tjk')
      }
    }

    stage('Terraform apply') {
      steps {
        withVault(configuration: [vaultUrl: 'https://dodt-vault.acldevsre.de',  vaultCredentialId: 'approle-for-vault', engineVersion: 2], vaultSecrets: [[path: 'jenkins/tjk', secretValues: [[envVar: 'AWS_ACCESS_KEY_ID', vaultKey: 'aws_access_key_id'],[envVar: 'AWS_SECRET_ACCESS_KEY', vaultKey: 'aws_secret_access_key'],[envVar: 'AWS_SESSION_TOKEN', vaultKey: 'aws_session_token']]]]) {
          sh '''
cd terraform/Target_infra
terraform apply -auto-approve
'''
        }

      }
    }

  }
  tools {
    terraform 'terraform-0.12.29'
  }
}