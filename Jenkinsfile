pipeline {

    agent any

    tools {
        maven "Maven 3.8.4"
    }

    environment {
        AWS_ACCOUNT_ID = credentials('AWS_ACCOUNT_ID')
        AWS_REGION = "us-west-1"
        REPO_NAME = "underwriter-microservice-rl"
    }

    stages {

            stage("Prepare") {

                steps {
                    sh "echo 'preparing...'"
                    sh 'git submodule init'
                    sh 'git submodule update'
                }

            }

            stage("Test") {

                steps {
                    sh "echo 'testing...'" //maybe implement SonarQ since mvn tests don't work?
                }
                
            }

            stage("Build") {

                steps {
                    sh "echo 'building...'"
                    sh 'mvn clean' 
                    sh 'mvn package -DskipTests'
                }

            }


            stage("Dockerize") {

                steps {
                    sh 'echo "creating image in $(pwd)..."'
                    sh 'docker build --file=new-Dockerfile-underwriter --tag="$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:$BUILD_ID" .'
                    sh "docker image ls"
                }

            }

            stage("Push") {

                steps {

                    sh "echo 'pushing to ECR...'"

                    withAWS(credentials: 'AWS_Ricky', region: 'us-west-1'){

                        sh 'aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com"'
                        sh 'docker image push "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:$BUILD_ID"'

                    }

                }

            }

    }

    post {

        always {
            sh 'docker rmi "$AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com/$REPO_NAME:$BUILD_ID"' 
        }

    }

}