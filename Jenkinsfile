def registry = 'https://sambooo.jfrog.io/'
def imageName = 'valaxy02.jfrog.io/valaxy-docker/ttrend'
def version   = '2.1.2'
pipeline {
    agent {
        node {
            label 'maven'
        }
    }

    environment {
        PATH = "/opt/apache-maven-3.9.6/bin:$PATH"
    }
    stages {
        stage('build') {
            steps {
                echo "------------ build started ---------"
                sh 'mvn clean deploy'
                echo "------------ build completed ---------"              
            }
        }
        stage("Jar Publish") {
            steps {
                script {
                    echo '<--------------- Jar Publish Started --------------->'
                    def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jfrog"
                    echo '<--------------- Jar Publish Started 2 --------------->'
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                    def uploadSpec = """{
                        "files": [
                            {
                            "pattern": "jarstaging/(*)",
                            "target": "libs-release-local/{1}",
                            "flat": "false",
                            "props" : "${properties}",
                            "exclusions": [ "*.sha1", "*.md5"]
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