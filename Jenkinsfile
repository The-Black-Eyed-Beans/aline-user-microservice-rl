pipeline {

    agent any

    tools {
        maven "Maven 3.8.4"
    }

    environment {
        AWS_ACCOUNT_ID = credentials('AWS_ACCOUNT_ID')
        AWS_REGION = "us-west-1"
        REPO_NAME = "user-microservice-rl"
    }

    stages {

            stage("Prepare") {

                steps {
                    sh "echo 'preparing...'"
                    sh 'git submodule init'
                    sh 'git submodule update'
                }

            }

            stage("Build") {

                steps {
                    sh "echo 'testing...'"
                    sh 'mvn clean' 
                    sh 'mvn package -DtestFailureIgnore=true'
                }
            }

            stage("Sonar Scan") {
                steps {

                    withSonarQubeEnv('SonarQube-Server') {
                        sh 'mvn sonar:sonar -Dsonar.projectName="$REPO_NAME"'

                    }
                }
            }

            stage("Quality Gate") {
                steps {

                    timeout(time: 5, unit: 'MINUTES'){
                        waitForQualityGate abortPipeline: true
                    }

                }
            }            


            stage("Dockerize") {

                steps {
                    sh 'echo "creating image in $(pwd)..."'
                    sh 'docker build --file=new-Dockerfile-user --tag="$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:$(git rev-parse HEAD)" .'
                    sh "docker image ls"
                }

            }

            stage("Push") {

                steps {

                    sh "echo 'pushing to ECR...'"

                    withAWS(credentials: 'AWS_Ricky', region: 'us-west-1'){

                        sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com"'
                        sh 'docker image push "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:$(git rev-parse HEAD)"'

                    }
                }
            }
    }

    post {

        always {
            sh 'docker rmi "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:$(git rev-parse HEAD)"' 
        }

    }

}