pipeline {
    agent {
        docker {
            image 'node:16-buster-slim'
            args '-p 3000:3000'
        }
    }
    tools {
        jdk 'jdk-11.0.16.101-hotspot'
        jdk 'Java SE Development Kit 9.0.4'
    }
    environment {
        CI = 'true'
        JAVA_HOME="${tool 'jdk-11.0.16.101-hotspot'}"
        PATH="${env.JAVA_HOME}/bin:${env.PATH}"
    }
    stages {
         stage('Configure') {
            steps {
                sh 'set NODE_OPTIONS=--openssl-legacy-provider'
                sh 'export NODE_OPTIONS=--openssl-legacy-provider'
                withSonarQubeEnv('SonarQube', envOnly: true) {
                  // This expands the evironment variables SONAR_CONFIG_NAME, SONAR_HOST_URL, SONAR_AUTH_TOKEN that can be used by any script.
                  println ${env.SONAR_HOST_URL} 
                }
                def scannerHome = tool 'SonarScanner 4.0';
                withSonarQubeEnv('SonarQube') { // If you have configured more than one global server connection, you can specify its name
                  sh "${scannerHome}/bin/sonar-scanner"
                }
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
            tools {
                jdk 'jdk-11.0.16.101-hotspot'
                jdk 'Java SE Development Kit 9.0.4'
            }
            environment {
                CI = 'true'
                JAVA_HOME="${tool 'jdk-11.0.16.101-hotspot'}"
                PATH="${env.JAVA_HOME}/bin:${env.PATH}"
            }
            steps {
                sh 'npm run test -- --coverage . --watchAll=false'
                sh 'chmod +x /var/jenkins_home/workspace/simple-node-js-react-npm-app/node_modules/sonar-scanner/bin/sonar-scanner'
                sh 'java --version'
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
