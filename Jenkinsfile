pipeline {
    agent {
        docker { 
            image 'node:16'
            args '-v /var/jenkins_home:/var/jenkins_home'
        }
    }
    stages {
        stage('Npm Install') {
            steps {
                echo "Npm install"
                sh 'npm install --save'
            }
        }
        stage('Unit test') {
            steps {
                echo "Unit test"
                sh 'npm test'
            }
        }
        stage('Vulnerability scan') {
            steps {
                echo "vulnerability scan"
                snykSecurity(
                  snykInstallation: 'snyk@latest',
                  snykTokenId: 'snyk-token',
                  failOnIssues: true,
                  failOnError: true,
                  severity: 'high'
                )
            }
        }
    }
}
