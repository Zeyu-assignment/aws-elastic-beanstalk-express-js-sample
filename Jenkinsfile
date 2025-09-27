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
                        sh 'mkdir -p /var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs'
                        echo "Npm install"
                        sh "npm install --save 2>&1 | tee /var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs/npm_install_${BUILD_NUMBER}.log"
                    }
                }
                
                stage('Unit test') {
                    steps {
                        echo "Unit test"
                        sh "npm test 2>&1 | tee /var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs/npm_test_${BUILD_NUMBER}.log"
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
                            def buildLog = "/var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs/docker_build_${BUILD_NUMBER}.log"
                            sh "docker build -t 21441677/assignment2_21441677:${BUILD_NUMBER} . 2>&1 | tee ${buildLog}"
                            sh "docker tag 21441677/assignment2_21441677:${BUILD_NUMBER} 21441677/assignment2_21441677:latest"
                        }
                    }
                }
                
                stage('Push image') {
                    steps {
                        echo "Push image"
                        script {
                            def pushLog = "/var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs/docker_push_${BUILD_NUMBER}.log"
                            docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                                sh "docker push 21441677/assignment2_21441677:${BUILD_NUMBER} 2>&1 | tee ${pushLog}"
                                sh "docker push 21441677/assignment2_21441677:latest 2>&1 | tee -a ${pushLog}"
                            }
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            node('built-in') {
                script {
                    sh "cp -r /var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs ${WORKSPACE}/"
                    sh "cp /var/jenkins_home/jobs/${env.JOB_NAME}/builds/${env.BUILD_NUMBER}/log ${WORKSPACE}/build.log"
                    archiveArtifacts artifacts: 'logs/*,build.log', allowEmptyArchive: true
                }
            }
        }
    }
}
