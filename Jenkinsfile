// Test3 03/27/2024
def registry = 'https://sambooo.jfrog.io/'
def imageName = 'sambooo.jfrog.io/samboo-docker-local/ttrend'
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

        // stage('SonarQube analysis') {
        //     environment {
        //         scannerHome = tool 'samboo-sonar-scanner'  // configured on Manage Jenkins under Tools
        //     }
        //     steps{
        //         withSonarQubeEnv('SonarQube') { // If you have configured more than one global server connection, you can specify its name
        //             sh "${scannerHome}/bin/sonar-scanner"
        //         }
        //     }
        // }

        // stage("Quality Gate"){
        //     steps {
        //         script {
        //             timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
        //             def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
        //                 if (qg.status != 'OK') {
        //                     error "Pipeline aborted due to quality gate failure: ${qg.status}"
        //                 }
        //             }
        //         }
        //     }
        // }

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
                            "target": "maven-libs-release-local/{1}/{1}",
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
        stage(" Docker Build ") {
            steps {
                script {
                echo '<--------------- Docker Build Started --------------->'
                app = docker.build(imageName+":"+version)
                echo '<--------------- Docker Build Ends --------------->'
                }
            }
        }
        stage (" Docker Publish "){
            steps {
                script {
                echo '<--------------- Docker Publish Started --------------->'  
                    docker.withRegistry(registry, 'jfrog-cred'){
                        app.push()
                    }    
                echo '<--------------- Docker Publish Ended --------------->'  
                }
            }
        }
        stage ("Deploy"){
            steps {
                script {
                    sh "chmod +x ./deploy.sh"
                    sh './deploy.sh'
                }
            }
        }

    }
}