pipeline {
    agent {
        docker {
            image 'node:16-buster-slim'
            args '-p 3000:3000'
        }
    }
    environment {
        CI = 'true'
    }
    stages {
         stage('Configure') {
            steps {
                sh 'set NODE_OPTIONS=--openssl-legacy-provider'
                sh 'export NODE_OPTIONS=--openssl-legacy-provider'
            }
        }
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                sh './jenkins/scripts/test.sh'
            }
        }
        stage('SonarQube analysis') {
            steps {
                sh 'npm run test -- --coverage . --watchAll=false'
                sh 'chmod +x /var/jenkins_home/workspace/simple-node-js-react-npm-app/node_modules/sonar-scanner/bin/sonar-scanner'
                withSonarQubeEnv('SonarQube') {
                    sh "npx sonar-scanner -D sonar.projectKey=React-SonarQube -D sonar.login=b082a592b5caae1791279ed841c00f3d865a24e4"
                }
            }
        }
        stage("Quality gate") {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }
        stage('Deliver') {
            steps {
                sh './jenkins/scripts/deliver.sh'
                input message: 'Finished using the web site? (Click "Proceed" to continue)'
                sh './jenkins/scripts/kill.sh'
            }
        }
    }
}
