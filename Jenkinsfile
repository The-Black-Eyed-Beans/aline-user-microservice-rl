pipeline {

    agent any

    tools {
        maven "Maven 3.8.4"
    }

    environment {
        AWS_ACCOUNT_ID = credentials('AWS_ACCOUNT_ID')
        JFROG_USER = credentials('JFROG_USER')
        JFROG_PW = credentials('JFROG_PW')
        JFROG_HOST = credentials('JFROG_HOST')
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
                sh 'mvn package -Dmaven.test.failure.ignore=true'
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
                sh 'docker build --file=new-Dockerfile-user --tag="$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:$(git rev-parse HEAD)" \
                --tag="$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:latest" \
                --tag="$JFROG_HOST/microservices/$REPO_NAME:$(git rev-parse HEAD)" \
                --tag="$JFROG_HOST/microservices/$REPO_NAME:latest" .'
                sh "docker image ls"
            }

        }

        stage("Push") {
            steps {
                withAWS(credentials: 'AWS_Ricky', region: 'us-west-1'){
                    sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com"'
                    sh 'docker image push "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:$(git rev-parse HEAD)"'
                    sh 'docker image push "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:latest"'
                }

                sh 'docker login -u $JFROG_USER -p $JFROG_PW $JFROG_HOST && \
                docker image push "$JFROG_HOST/microservices/$REPO_NAME:$(git rev-parse HEAD)" && \
                docker image push "$JFROG_HOST/microservices/$REPO_NAME:latest"'

                sh 'curl -X "PUT" -u "$JFROG_USER:$JFROG_PW" --upload-file "$REPO_NAME-0.1.0.jar" "http://$JFROG_HOST/artifactory/microservices-jar-local/$REPO_NAME-0.1.0.jar"'

            }
        }
    }

    post {
        always {
            sh 'docker rmi "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:$(git rev-parse HEAD)"'
            sh 'docker rmi "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:latest"' 
            sh 'docker rmi "$JFROG_HOST/microservices/$REPO_NAME:$(git rev-parse HEAD)"' 
            sh 'docker rmi "$JFROG_HOST/microservices/$REPO_NAME:latest"'
        }

    }

}