pipeline {
    agent none
    
    stages {
        stage('Npm Tasks') {
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
        
        stage('Docker build and push') {
            agent any  
            stages {
                stage('Build image') {
                    steps {
                        echo "Build image"
                        script {
                            app = docker.build("21441677/assignment2_21441677")
                        }
                    }
                }
                
                stage('Push image') {
                    steps {
                        echo "Push image"
                        script {
                            docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                                app.push("${env.BUILD_NUMBER}")
                                app.push("latest")
                            }
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            node {
                archiveArtifacts artifacts: 'log', allowEmptyArchive: true
            }
        }
    }
}
