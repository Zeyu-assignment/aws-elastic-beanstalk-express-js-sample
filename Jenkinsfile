pipeline {
    agent {
        docker { image 'node:16' }
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
    }
}
