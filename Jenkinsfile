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
cp /var/jenkins_home/terraform.tar .
tar xvf terraform.tar
'''
        sh '''
whoami
pwd
            KUBECTL=./kubectl
            if [ -f "$KUBECTL" ]; then
              echo "$KUBECTL exists"
            else
              curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
              chmod +x ./kubectl
              mv ./kubectl $KUBECTL
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