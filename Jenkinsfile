pipeline {
    agent {
        docker { 
            image 'node:16'
            args '-v /var/run/docker.sock:/var/run/docker.sock -v /usr/local/bin/docker:/usr/local/bin/docker -v /var/jenkins_home:/var/jenkins_home'
        }
    }
    
    stages {
        stage('Npm install') {
            steps {
                sh "mkdir -p /var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs"
                sh "echo 'Npm install......' | tee /var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs/npm_install_${BUILD_NUMBER}.log"
                sh "npm install --save >> /var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs/npm_install_${BUILD_NUMBER}.log 2>&1"
                sh "cat /var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs/npm_install_${BUILD_NUMBER}.log"
            }
        }
        
        stage('Unit tests') {
            steps {
                sh "echo 'Unit tests......' | tee /var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs/unit_tests_${BUILD_NUMBER}.log"
                sh "npm test >> /var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs/unit_tests_${BUILD_NUMBER}.log 2>&1"
                sh "cat /var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs/unit_tests_${BUILD_NUMBER}.log"
            }
        }
        
        stage('Vulnerability scan') {
            steps {
                echo "Vulnerability scan......"
                snykSecurity(
                  snykInstallation: 'snyk@latest',
                  snykTokenId: 'snyk-token',
                  failOnIssues: true,
                  failOnError: true,
                  severity: 'high'
                )
            }
        }
        
        stage('Docker Build') {
            steps {
                script {
                    def buildLog = "/var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs/docker_build_${BUILD_NUMBER}.log"
		    sh "echo 'Build image......' |tee ${buildLog}"
                    sh "docker build -t 21441677/assignment2_21441677:${BUILD_NUMBER} . >> ${buildLog} 2>&1"
                    sh "docker tag 21441677/assignment2_21441677:${BUILD_NUMBER} 21441677/assignment2_21441677:latest >> ${buildLog} 2>&1"
                    sh "cat ${buildLog}"
                }
            }
        }
        
        stage('Docker Push') {
            steps {
                script {
                    def pushLog = "/var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs/docker_push_${BUILD_NUMBER}.log"
		    sh "echo 'Push image......' |tee ${pushLog}"
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin >> ${pushLog} 2>&1"
                        sh "docker push 21441677/assignment2_21441677:${BUILD_NUMBER} >> ${pushLog} 2>&1"
                        sh "docker push 21441677/assignment2_21441677:latest >> ${pushLog} 2>&1"
                        sh "cat ${pushLog}"
                    }
                }
            }
        }
    }

    post {
        always {
            sh "cp -r /var/jenkins_home/jobs/${env.JOB_NAME}/${BUILD_NUMBER}/logs ${WORKSPACE}/"
            sh "cp /var/jenkins_home/jobs/${env.JOB_NAME}/builds/${env.BUILD_NUMBER}/log ${WORKSPACE}/pipeline.log"
            archiveArtifacts artifacts: "logs/*${BUILD_NUMBER}.log,pipeline.log", allowEmptyArchive: true
        }
    }
}
