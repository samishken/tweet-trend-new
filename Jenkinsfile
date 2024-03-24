def registry = 'https://sambooo.jfrog.io/'
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
                    def server = Artifactory.newServer url:registry+"/artifactory" ,  credentialsId:"jfrog-cred"
                    echo '<--------------- Jar Publish Started 2 --------------->'
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                    def uploadSpec = """{
                        "files": [
                            {
                            "pattern": "jarstaging/(*)",
                            "target": "ttrend-libs-release-local/{1}/{1}",
                            "flat": "false",
                            "props" : "${properties}",
                            "exclusions": [ "*.sha1", "*.md5"]
                            }
                        ]
                    }"""
                    echo '<--------------- Jar Publish Started 3 --------------->'
                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                    echo '<--------------- Jar Publish Ended --------------->'  
                
                }
            }   
        }
    }
}