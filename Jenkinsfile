def registry = 'https://mask9147.jfrog.io'
def imageName = 'mask9147.jfrog.io/mask9147-docker-local/sample_app'
def version = '2.1.2'

pipeline {
    agent {
        node {
            label 'maven'
        }
    }
    environment {
        PATH = "/opt/apache-maven-3.9.4/bin:$PATH"
        // Artifactory server details
        ARTIFACTORY_SERVER = 'artifactory-server_id' // Replace with your Artifactory server ID in Jenkins
        ARTIFACTORY_URL = 'https://mask9147.jfrog.io/artifactory' // Replace with your Artifactory URL
        ARTIFACTORY_REPO = 'mavenrepo-libs-release' // Replace with your target repository
        ARTIFACTORY_CREDENTIALS = 'JFROG_ARTIFACORY_UI' // Replace with your Jenkins credential ID    
    }
    stages {
        stage("build") {
            steps {
                echo "----------- build started ----------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "----------- build completed ----------"
            }
        }
        stage("test") {
            steps {
                echo "----------- unit test started ----------"
                sh 'mvn surefire-report:report'
                echo "----------- unit test completed ----------"
            }
        }
        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'SonarQubeScanner5'
            }
            steps {
                withSonarQubeEnv('SonarQubeScanner3') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        // Just in case something goes wrong, pipeline will be killed after a timeout
                        def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
        stage("Jar Publish") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'
                    def server = Artifactory.server(ARTIFACTORY_SERVER)
                    server.url = ARTIFACTORY_URL
                    server.credentialsId = ARTIFACTORY_CREDENTIALS

                    def properties = "buildid=${env.BUILD_ID},commitid=${env.GIT_COMMIT}"
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/(*)",
                                "target": "${ARTIFACTORY_REPO}/{1}",
                                "flat": false,
                                "props": "${properties}",
                                "exclusions": [ "*.sha1", "*.md5" ]
                            }
                        ]
                    }"""
                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                    echo '<--------------- Jar Publish Ended --------------->'
                }
            }
        }
    }
}
