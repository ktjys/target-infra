pipeline {
  agent {
    kubernetes {
      yaml '''
kind: Pod
metadata:
  name: target-infra
  labels:
    app: jenkins-slave
spec:
  containers:
  - name: awscli
    image: amazon/aws-cli:latest
    command:
    - /bin/sh
    tty: true
  - name: helm
    image: alpine/helm:latest
    command:
    - /bin/sh
    tty: true
'''
      defaultContainer 'jnlp'
    }

  }
  stages {
    stage('Prepare') {
      steps {
        sh '''
curl https://nexus.acldevsre.de/repository/target-infra/target-infra/target-infra-tar/v0.0.1/target-infra-tar-v0.0.1.tar -o terraform.tar 
tar xvf terraform.tar
sed -i "s/eks_public_access_cidrs.*/eks_public_access_cidrs= [\\"15.164.177.114\\/32\\",\\"3.35.17.6\\/32\\",\\"13.125.181.170\\/32\\",/" ./terraform/Target_infra/terraform.auto.tfvars
'''
      }
    }

    stage('Terraform apply') {
      steps {
        withVault(configuration: [vaultUrl: 'https://dodt-vault.acldevsre.de',  vaultCredentialId: 'approle-for-vault', engineVersion: 2], vaultSecrets: [[path: 'jenkins/tjk', secretValues: [[envVar: 'AWS_ACCESS_KEY_ID', vaultKey: 'aws_access_key_id'],[envVar: 'AWS_SECRET_ACCESS_KEY', vaultKey: 'aws_secret_access_key'],[envVar: 'AWS_SESSION_TOKEN', vaultKey: 'aws_session_token']]]]) {
          sh '''
cd terraform/Target_infra
terraform init
terraform apply -auto-approve
'''
        }

      }
    }

    stage('Get Kubeconfig') {
      steps {
        container(name: 'awscli') {
          withVault(configuration: [vaultUrl: 'https://dodt-vault.acldevsre.de',  vaultCredentialId: 'approle-for-vault', engineVersion: 2], vaultSecrets: [[path: 'jenkins/tjk', secretValues: [[envVar: 'AWS_ACCESS_KEY_ID', vaultKey: 'aws_access_key_id'],[envVar: 'AWS_SECRET_ACCESS_KEY', vaultKey: 'aws_secret_access_key'],[envVar: 'AWS_SESSION_TOKEN', vaultKey: 'aws_session_token']]]]) {
            sh '''
                cd terraform/Target_infra

mkdir ~/.aws -p
AWS_PROFILE=$(cat terraform.auto.tfvars | awk \'/aws_profile/ {print $3}\' | tr -d "\\"")
echo $AWS_PROFILE

cat > ~/.aws/config << EOF
[profile $AWS_PROFILE]
region = ap-northeast-2
output = json
EOF

cat << EOF > ~/.aws/credentials
[$AWS_PROFILE]
aws_access_key_id=$AWS_ACCESS_KEY_ID
aws_secret_access_key=$AWS_SECRET_ACCESS_KEY
aws_session_token=$AWS_SESSION_TOKEN
EOF

            KUBECTL=/usr/local/bin/kubectl
            if [ -f "$KUBECTL" ]; then
              echo "$KUBECTL exists"
            else
              mkdir ~/bin -p
              curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
              chmod +x ./kubectl
              mv ./kubectl $KUBECTL
              kubectl version --short --client
            fi

                EKS_CLUSTER=`terraform output | awk \'/cluster_name/ {print $3}\'`
                echo "cluster_name = $EKS_CLUSTER" > output.tmp
                cd $EKS_CLUSTER
                ./1.update-kubeconfig.sh
                
                KUBECONFIG_PATH=~/.kube
                if [ -d "$KUBECONFIG_PATH" ]; then
                    echo "$KUBECONFIG_PATH exist"
                else
                    mkdir $KUBECONFIG_PATH
                fi
                
                cp config-$EKS_CLUSTER ~/.kube/config
cat ~/.kube/config
                kubectl get ns

kubectl set env daemonset -n kube-system aws-node AWS_VPC_K8S_CNI_EXTERNALSNAT=true
#kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller//crds?ref=master"

'''
          }

        }

      }
    }

    stage('Install alb ingress controller') {
      steps {
        container(name: 'helm') {
          dir(path: './terraform/Target_infra') {
            sh '''

                  EKS_CLUSTER=`cat output.tmp | awk \'/cluster_name/ {print $3}\'`
                  echo "$EKS_CLUSTER"
                  EKS_REPO=`helm repo list|awk \'/eks/\' | sed \'s/ *$//g\'`
                  if [ ! "$EKS_REPO" ]; then
                    echo "eks helm repo not exist"
                    helm repo add eks https://aws.github.io/eks-charts
                    helm repo update
                  fi
                  
                KUBECONFIG_PATH=~/.kube
                if [ -d "$KUBECONFIG_PATH" ]; then
                    echo "$KUBECONFIG_PATH exist"
                else
                    mkdir $KUBECONFIG_PATH
                fi
                
                cp config-$EKS_CLUSTER ~/.kube/config

                  ALB_REL=`helm ls -n kube-system | awk \'/aws-load-balancer-controller/\' | sed \'s/ *$//g\'`
                  if [ ! "$ALB_REL" ]; then
                    helm upgrade -i aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=$EKS_CLUSTER
                  fi   
              '''
          }

        }

      }
    }

  }
  tools {
    terraform 'terraform-0.12.29'
  }
}