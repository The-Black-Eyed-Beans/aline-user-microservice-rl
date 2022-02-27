pipeline {

    agent any

    tools {
        maven "Maven"
    }

    stages {

            stage("prepare") {

                steps {
                    sh "echo 'preparing..."
                    sh 'git submodule init'
                    sh 'git submodule update'
                }

            }

            stage("test") {

                steps {
                    sh "echo 'testing..."
                    sh 'mvn clean test' 
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
                    sh "echo 'creating image...'"
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