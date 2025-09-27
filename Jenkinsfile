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
                        sh "set -e && mkdir -p /var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs"
                        echo "Npm install"
                        sh "set -e && npm install --save > /var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs/npm_install_${BUILD_NUMBER}.log 2>&1"
                        sh "cat /var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs/npm_install_${BUILD_NUMBER}.log"
                    }
                }
                
                stage('Unit test') {
                    steps {
                        echo "Unit test"
                        sh "set -e && npm test > /var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs/npm_test_${BUILD_NUMBER}.log 2>&1"
                        sh "cat /var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs/npm_test_${BUILD_NUMBER}.log"
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
                            sh "set -e && docker build -t 21441677/assignment2_21441677:${BUILD_NUMBER} . > ${buildLog} 2>&1"
                            sh "cat ${buildLog}"
                            sh "set -e && docker tag 21441677/assignment2_21441677:${BUILD_NUMBER} 21441677/assignment2_21441677:latest"
                        }
                    }
                }
                
                stage('Push image') {
                    steps {
                        echo "Push image"
                        script {
                            def pushLog = "/var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs/docker_push_${BUILD_NUMBER}.log"
                            withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                                sh "set -e && echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin > ${pushLog} 2>&1"
                                sh "set -e && docker push 21441677/assignment2_21441677:${BUILD_NUMBER} >> ${pushLog} 2>&1"
                                sh "set -e && docker push 21441677/assignment2_21441677:latest >> ${pushLog} 2>&1"
                                sh "cat ${pushLog}"
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
