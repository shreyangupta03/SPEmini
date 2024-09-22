pipeline {
    
    agent {
        any {
            label 'docker'
            image 'node:6-alpine'
            args '-p 3000:3000'
        }
    }
    
     environment {
            CI = 'true'
            registry = 'sathvik04/scientific-calculator'
            DOCKERHUB_CRED = credentials('dockerhub_id')
            registryCredential = 'dockerhub_id'
            dockerimage = ''
      }
    stages {
        stage('Git Pull') {
            steps {
                git url: 'https://github.com/sathvik-bhat/Scientific-Calculator.git', branch: 'master',
                credentialsId: 'Credential_Git'
            }
        }
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'tar czf Node.tar.gz node_modules src jenkins Jenkinsfile package.json public'
            }
        }

        stage('Test') {
            steps {
                sh 'chmod +x ./jenkins/scripts/test.sh'
                sh './jenkins/scripts/test.sh'
            }
        }
        stage('Build Docker Image') {

            steps {
                script{
                    dockerimage = sh 'docker build -t '+registry+':latest .'
                }
            }
        }
        stage('Push Image to dockerHub') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub_pwd', variable: 'dockerhub_pwd')]) {
                    sh 'docker login -u "sathvik04" -p ${dockerhub_pwd}'
                    sh 'docker push ' +registry +':latest'
                }
            }
        }
        stage('Free local space') {
            steps {
                sh 'docker rmi $registry:latest'
            }
        }

        stage('Deploy') {
            steps {
                sh 'chmod 600 scientific-calculator.pem'
                ansiblePlaybook becomeUser: null, colorized: true, disableHostKeyChecking: true, installation: 'ansible', inventory: 'inventory',
                playbook: 'playbook.yml', sudoUser: null, extras: '-e "image_name=sathvik04/scientific-calculator"'
            }
        }
    }
}  