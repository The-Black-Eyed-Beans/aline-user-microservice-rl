pipeline {

    agent any

    tools {
        maven "Maven 3.8.4"
    }

    stages {

            stage("prepare") {

                steps {
                    sh "echo 'preparing...'"
                    sh 'git submodule init'
                    sh 'git submodule update'
                }

            }

            stage("test") {

                steps {
                    sh "echo 'testing...'"
                    sh 'mvn clean' 
                    sh 'mvn package -DskipTests'
                }

            }

            stage("build") {

                steps {
                    sh "echo 'building...'"
                }

            }

            stage("create image") {

                steps {
                    sh 'echo "creating image in $(pwd)..."'
                    sh "docker build --file=new-Dockerfile-user --tag=user-rl:latest"
                    sh "docker image ls"
                }

            }

            stage("push to ECR") {

                steps {
                    sh "echo 'pushing to ECR...'"
                }

            }

    }

    post {

        success {
            sh "echo 'Success!'"
        }

    }

}