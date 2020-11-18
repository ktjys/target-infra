pipeline {
  agent any
  stages {
    stage('Prepare') {
      steps {
        sh '''ls /var/jenkins_home/userContent
cp /var/jenkins_home/userContent/target.tar .
tar xvf target.tar
'''
      }
    }

    stage('Terraform apply') {
      steps {
        withVault(configuration: [vaultUrl: 'https://dodt-vault.acldevsre.de',  vaultCredentialId: 'approle-for-vault', engineVersion: 2], vaultSecrets: [[path: 'jenkins/tjk', secretValues: [[envVar: 'AWS_ACCESS_KEY_ID', vaultKey: 'aws_access_key_id'],[envVar: 'AWS_SECRET_ACCESS_KEY', vaultKey: 'aws_secret_access_key'],[envVar: 'AWS_SESSION_TOKEN', vaultKey: 'aws_session_token']]]]) {
          sh '''

echo $AWS_ACCESS_KEY_ID
echo $AWS_SECRET_ACCESS_KEY
echo $AWS_SESSION_TOKEN
cd Target_infra
ls
terraform init
terraform plan
'''
        }

      }
    }

  }
  tools {
    terraform 'terraform-0.12.29'
  }
}