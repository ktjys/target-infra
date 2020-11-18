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

export AWS_ACCESS_KEY_ID=ASIAV2ROTHZK54QK7J73
export AWS_SECRET_ACCESS_KEY=SlWYKr4jM5+k/IISqZcGy0mPbp6P4rmz/RpNd+nn
export AWS_SESSION_TOKEN=IQoJb3JpZ2luX2VjEMj//////////wEaDmFwLW5vcnRoZWFzdC0yIkYwRAIgXPdLANGBwszt+a7cTAbgRaALihSY871L3CkFYERo7J8CIE42O0OWTNKPilEfNWGPBT5aH7x1hUkLI/p6tBJYiaeUKu8BCEEQAxoMNDAwNjAzNDMwNDg1IgzhGsz+UpbAnu5Ih0cqzAG2zTKsqOePoVTH+f379pMLtMTnY887mztQEto4tSn8vGDH4QsdxdEPgsTgca4fyh1xP/NLxc3iusikgrTQpG2L/JGgE35WSY4BYj2jQgjSU3ZvUqmWkaM/GqjjxZDL+5Sd0wnh9iQ2DE4wzVKgH75aDPkpOcVTmwtF3UBZhGVpaA04n39Ip/jeF3+3/N6Ta4qfvr9FpDUfU6vq/ZRe/Lb9AW4xyMFm9nfokCusZkIQqCDA3mIpxHLzSR/VolDlAx1DhTsstR2QkmZ5Yocw9KrT/QU6mQFIy9pxBe8GzIuwihO/Plq3y/sBTqsQvy7zTfkLwXJ7OKoA/l/i9QgJzzvJ7XrEho+mT3xdJz2sa5Ny7FQkbL1/t/v1q+Y30xSdIwgO23FUH14k00gaz06r9wQq9vpilgPRRkkrsGH7nq6f/pnOlAx+GoaiWPkwHXKeGb2XBW+VKH4lTAdFglOT8R+jqQcKIbVfdzbvHVd4UQk=

echo $AWS_ACCESS_KEY_ID
echo $AWS_SECRET_ACCESS_KEY
echo $AWS_SESSION_TOKEN
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