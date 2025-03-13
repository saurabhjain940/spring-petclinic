pipeline {
    agent {
        docker { 
            alwaysPull true
            image '730335384723.dkr.ecr.ap-south-1.amazonaws.com/jenkins/agent:latest'
            registryUrl 'https://730335384723.dkr.ecr.ap-south-1.amazonaws.com'
            registryCredentialsId 'ecr:ap-south-1:aws'
            args '-v /var/run/docker.sock:/var/run/docker.sock -u 0:0'
            }
    }

    stages {

        stage('Checkout Code') {
            steps {
                // Get the latest code from your version control system
                checkout scm
            }
        }
        stage('Install dependencies') {
            steps {
                echo 'Install..'
            }
        }
        stage('Build') {
            steps {
                echo 'Building..'
                sh 'docker build -t petclinic:latest .'
            }
        }
        stage('Push') {
            steps {
                echo 'Building..'
                sh '''
                aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 730335384723.dkr.ecr.ap-south-1.amazonaws.com
                docker tag petclinic:latest 730335384723.dkr.ecr.ap-south-1.amazonaws.com/hybrid:latest
                docker push 730335384723.dkr.ecr.ap-south-1.amazonaws.com/hybrid:latest
                '''
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
