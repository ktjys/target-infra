pipeline {
  agent any
  stages {
    stage('Prepare') {
      steps {
        sh '''ls /var/jenkins_home/userContent
cp /var/jenkins_home/userContent/target.tar .
tar xvf target.tar
'''
        stash 'ARTIFACTS'
      }
    }

    stage('Terraform apply') {
      agent {
        kubernetes {
          yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: amazon/aws-cli:latest
    command:
    - /bin/sh
    tty: true
'''
          defaultContainer 'shell'
        }

      }
      steps {
        unstash 'ARTIFACTS'
        withVault(configuration: [vaultUrl: 'https://dodt-vault.acldevsre.de',  vaultCredentialId: 'approle-for-vault', engineVersion: 2], vaultSecrets: [[path: 'jenkins/tjk', secretValues: [[envVar: 'AWS_ACCESS_KEY_ID', vaultKey: 'aws_access_key_id'],[envVar: 'AWS_SECRET_ACCESS_KEY', vaultKey: 'aws_secret_access_key'],[envVar: 'AWS_SESSION_TOKEN', vaultKey: 'aws_session_token']]]]) {
          sh '''

export AWS_ACCESS_KEY_ID=ASIAV2ROTHZKWPREYNXE
export AWS_SECRET_ACCESS_KEY=vvdddcrmrpI1yDceJBfVTPX27rDlT2QTXeoE+jAb
export AWS_SESSION_TOKEN=IQoJb3JpZ2luX2VjENn//////////wEaDmFwLW5vcnRoZWFzdC0yIkYwRAIgaMeT9bCookzFekU23lL9hcjAo63smlaiZSUZI6gBN2ECIEFru/ZMVf9TCeO4FpFzvlKflNS+Bw/x+oT0ZZlx+ZHRKu8BCFIQAxoMNDAwNjAzNDMwNDg1IgxJiWJqBwKyVapvUDgqzAFPD745cENe5c4DWg+jXExRPRagdcEN3eDib8EW0kMSRT2TRg31wPWT86uBOMdnjaPTPDcpd9OkzAqnUbpIHBTL6NnIma+w3YP3PdqhDMfUMmW/iH/OpWmwL2kcvFBKWPgHdJx3/mt6mRbnhNfjUo6zfKna8LGEj0+ZfSyf4HD+mZ8l67j0PlQhVLEr0YQ+VPEfd1ecTtOAm8EyTiqxIyFYCr2V2wThClkUvd9zrYcK8I0axxwToiJKK7BU2IvC6RdsB+FXlxFqiTCc2HUw9v/W/QU6mQEb7u3APOKZtGTi1pz6mVurGB/dUeC1a0w4c20xSuPeSkMaxr703bqT46ODRhrl6TtzNa46WdOb4n1JHNjzmC8zxbuTsrLt+gjyPVns3OVMVAAON1PmjyQlIJJm1baDZqNvzigWQbNjv7cLWqQ9kJQnUw8bUEyuKfcgeKlIqS+Mpvy/yjC1kOv6sjw1UXX35tGealtpY8Hh2aA=


aws ec2 describe-instances --filters "Name=instance-type,Values=t2.micro" --query "Reservations[].Instances[].InstanceId"
aws s3 ls

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