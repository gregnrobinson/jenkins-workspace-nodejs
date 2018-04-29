pipeline {
  agent {
    docker {
      image 'gregnrobinson/build-container:latest'
    }
  }
  stages {
    stage('Build') {
      steps {
        sh 'npm install'
      }
    }
    stage('Create Packer AMI') {
        steps {
          withCredentials([
            usernamePassword(credentialsId: '8af0770f-2748-4c91-b8d1-514d71ca8667', passwordVariable: 'AWS_SECRET', usernameVariable: 'AWS_KEY')
          ]) {
            sh 'packer build -var aws_access_key=${AWS_KEY} -var aws_secret_key=${AWS_SECRET} ./packer.json'
        }
      }
    }
    stage('AWS Deployment') {
      steps {
          withCredentials([
            usernamePassword(credentialsId: '8af0770f-2748-4c91-b8d1-514d71ca8667', passwordVariable: 'AWS_SECRET', usernameVariable: 'AWS_KEY'),
            usernamePassword(credentialsId: '212b50ff-5073-4d82-99e6-fbd67501543b', passwordVariable: 'REPO_PASS', usernameVariable: 'REPO_USER'),
          ]) {
            sh 'rm -rf node-app-terraform'
            sh 'git clone https://github.com/gregnrobinson/node-app-terraform.git'
            sh '''
               cd node-app-terraform
               terraform init
               terraform apply -auto-approve -var access_key=${AWS_KEY} -var secret_key=${AWS_SECRET}
               git add terraform.tfstate
               git -c user.name="Greg Robinson" -c user.email="gregnrobinsongmail.com" commit -m "terraform state update from Jenkins"
               git push https://${REPO_USER}:${REPO_PASS}@github.com/gregnrobinson/node-app-terraform.git master
            '''
        }
      }
    }
  }
  environment {
    npm_config_cache = 'npm-cache'
  }
}
