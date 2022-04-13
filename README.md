# testDocker

def img
pipeline {
    environment {
        registry = "rushabhgediya/django-jenkins"
        registryCredentials = "docker-hub-login"
        dockerImage = ''
        // FULL_PATH_BRANCH = "${sh(script:'git name-rev --name-only HEAD', returnStdout: true)}"
        // GIT_BRANCH = FULL_PATH_BRANCH.substring(FULL_PATH_BRANCH.lastIndexOf('/') + 1, FULL_PATH_BRANCH.length())
    }
    agent any

    stages {
        // stage('Pulling From') {
        //     steps {
        //         script {
        //             echo 'Pulling... ' + env.GIT_BRANCH
        //             println("${GIT_BRANCH}")
        //         }
        //     }
        // }
        
        stage('build checkout') {
            steps {
                git 'https://github.com/rushabhgediya38/testDocker.git'
            }
        }
        
        stage ('Build Docker Image') {
            steps {
                script {
                    img = registry + ":${env.BUILD_ID}"
                    println("${img}")
                    dockerImage = docker.build("${img}")
                }
                
            }
        }
        
        stage('docker-compose') {
           steps {
              sh "docker-compose build"
              sh "docker-compose up -d"
           }
       }
        
        // stage ('Testing - Django in jenikns node') {
        //     steps {
        //         sh "docker run -d --name ${JOB_NAME} -p 8000:8000 ${img}"
        //     }
        // }
        
        stage ('permission stage'){
            steps {
                sh "chmod -R a+rwx ./data"
            }
        }
        
        stage ('Push to docker hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', registryCredentials) {
                        dockerImage.push()
                    }
                }
            }
            
        }
    }
}
