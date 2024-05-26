pipeline {
    agent {
        docker {
            image 'maven:latest'
            args '-u root' // Running as root to avoid permission issues
        }
    }
    stages {
        stage('Sonar Scan') {
            steps {
                withSonarQubeEnv('sonarserver') { 
                    sh "mvn sonar:sonar"
                }
            }  
        }
        stage('Maven Build') {
            steps {
                sh "mvn clean install"
            }  
        }
        stage('Deploy') {
            steps {
                deploy adapters: [
                    tomcat8(
                        credentialsId: 'tomcat-deployer', 
                        path: '', 
                        url: 'http://100.26.59.164:8080/'
                    )
                ], contextPath: 'demo', war: 'target/*.war'
            }
        }
    }
}
