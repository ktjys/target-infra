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
export AWS_ACCESS_KEY_ID="ASIAV2ROTHZKRWDC5TVV"
export AWS_SECRET_ACCESS_KEY="GEul9zzvJd3QJIXsyG/aCWT9CIarO5+wQQTI39K/"
export AWS_SESSION_TOKEN="IQoJb3JpZ2luX2VjEMb//////////wEaDmFwLW5vcnRoZWFzdC0yIkgwRgIhAKDmnkZAT1rJhGF71XD82Hs2fyODiNTSmZ4yLT3e3gasAiEAtAsWueqie9vC4/iMCZKib0p+Y1UhJyo9puCalDwEA8wq7wEIPxADGgw0MDA2MDM0MzA0ODUiDJov7dfvX9Z2FSLrvirMAR57XH0VSVipuyS0weR5GKZMIU4w8mfdCHvsCxY8QfOU91uupMelysr6mWyrpHydILkYz5O4e6l0943XNSOBE+O9wnm3zxqx7mTpH2DPTGK84aaNZVKmPAQxk6gFQ2OLTguhjbiaxW/Dv+EYm6y/ztUEdWSv25OIH5bfefJR7o+a97XNg1TdH5rR7cnnt1ede6sC2msaWSiVqm2AkW+7Jt6MV6E60JgVWWwV5DZscDC0pe+8sf8VDfdz+g9y5S43uLHAjvmpggj+B72j3jD69dL9BTqXAQ3at1i4uiCUZmB1WN8eabnDPux291Q3w58gfObUwOtm5Tbg8FhGQHFWGd4eRLcrFmTQD6YwIQbk2zFyU1mV2hQ1qdS8Cxfj9vmqf4JeslujLjwokm+Qf0/XTC3zsRIY1cSDC0NXnNc0RjDvWgwbhXGZR8RYnjMGgLLVQZYCD+2cVyDnDlMFQFXaSWGAtqyrfxryCZF3PYM="
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