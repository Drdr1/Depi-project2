pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        sh 'docker build -t argon-dashboard-flask .'
        sh 'docker tag argon-dashboard-flask $DOCKER_BFLASK_IMAGE'
      }
    }
    stage('Test') {
      steps {
        sh 'docker run my-flask-app python -m pytest tests/test_app.py'
      }
    }
    stage('Deploy') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
          sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
          sh 'docker push $DOCKER_BFLASK_IMAGE'
        }
      }
    }
    stage('Run Ansible Playbook') {
      steps {
        script {
           ansiblePlaybook credentialsId: 'private_key', disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory.ini', playbook: 'cloud.yml', vaultTmpPath: ''
        }
      }
    }
  }
  post {
    always {
      sh 'docker logout'
    }
  }
}
