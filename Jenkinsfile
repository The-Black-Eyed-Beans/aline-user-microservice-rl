pipeline {

    agent any

    tools {
        maven "Maven 3.8.4"
    }

    environment {
        AWS_ACCOUNT_ID = credentials('AWS_ACCOUNT_ID')
        REPO_NAME = "user-microservice-rl"
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
                    sh 'docker build --file=new-Dockerfile-user --tag="$REPO_NAME:$BUILD_ID" .'
                    sh "docker image ls"
                }

            }

            stage("push to ECR") {

                steps {

                    sh "echo 'pushing to ECR...'"

                    withAWS(credentials: 'AWS_Ricky', region: 'us-west-1'){

                        sh 'aws ecr get-login-password --region us-west-1 | docker login --username AWS --password-stdin "$AWS_ACCOUNT_ID.dkr.ecr.us-west-1.amazonaws.com"'
                        sh 'docker image push "$AWS_ACCOUNT_ID.dkr.ecr.us-west-1.amazonaws.com/$REPO_NAME:$BUILD_ID"'

                    }
                    
                    // script {
                    //     docker.withRegistry("${env.AWS_ACCOUNT_ID}.dkr.ecr.us-west-1.amazonaws.com/user-microservice-rl", 'ecr:us-west-1:AWS_Ricky') {
                    //         docker.image("user-microservice-rl:${env.BUILD_ID}").push()
                    //     }
                    // }

                }

            }

    }

    post {

        success {
            sh "echo 'Success!'"
            sh 'docker rmi "$REPO_NAME:$BUILD_ID"' 
        }

        failure {
            sh 'docker rmi "$REPO_NAME:$BUILD_ID"'
        }

    }

}