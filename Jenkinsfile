def builderImage
def productionImage
def ACCOUNT_REGISTRY_PREFIX
def GIT_COMMIT_HASH


pipeline {
    agent any
    stages {
        stage('Checkout Source Code and Logging Into Registry') {
            steps {
                echo 'Logging Into the Private ECR Registry'
                script {
                    GIT_COMMIT_HASH = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true)
                    ACCOUNT_REGISTRY_PREFIX = "321442529690.dkr.ecr.us-west-2.amazonaws.com"
                    sh """
                    \$(aws ecr get-login --no-include-email --region us-west-2)
                    """
                }
            }
        }

        stage('Make A Builder Image') {
            steps {
                echo 'Starting to build the project builder docker image'
                script {
                    builderImage = docker.build("${ACCOUNT_REGISTRY_PREFIX}/example-webapp-builder:${GIT_COMMIT_HASH}", "-f ./Dockerfile.builder .")
                    builderImage.push()
                    builderImage.push("${env.GIT_BRANCH}")
                    builderImage.inside('-v $WORKSPACE:/output -u root') {
                        sh """
                           cd /output
                           lein uberjar
                        """
                    }
                }
            }
        }

        stage('Unit Tests') {
            steps {
                echo 'running unit tests in the builder image.'
                script {
                    builderImage.inside('-v $WORKSPACE:/output -u root') {
                    sh """
                       cd /output
                       lein test
                    """
                    }
                }
            }
        }

        stage('Build Production Image') {
            steps {
                echo 'Starting to build docker image'
                script {
                    productionImage = docker.build("${ACCOUNT_REGISTRY_PREFIX}/example-webapp:${GIT_COMMIT_HASH}")
                    productionImage.push()
                    productionImage.push("${env.GIT_BRANCH}")
                }
            }
        }

 
        stage('Deploy to Production fixed server') {
            when {
                branch 'master'
            }
            steps {
                echo 'Deploying release to production'
                script {
                    productionImage.push("deploy")
                   sh """
                       aws ec2 reboot-instances --region us-west-2 --instance-ids i-0f1a736171fc66557
                    """
                }
            }
        }


        stage('Integration Tests') {
            when {
                branch 'master'
            }
            steps {
                echo 'Deploy to test environment and run integration tests'
//                script {
//                    TEST_ALB_LISTENER_ARN="arn:aws:elasticloadbalancing:us-east-1:089778365617:listener/app/testing-website/3a4d20158ad2c734/49cb56d533c1772b"
//                    sh """
//                    ./run-stack.sh example-webapp-test ${TEST_ALB_LISTENER_ARN}
//                    """
//                }
                echo 'Running tests on the integration test environment'
//                script {
//                    sh """
//                       curl -v http://testing-website-1317230480.us-east-1.elb.amazonaws.com | grep '<title>Welcome to example-webapp</title>'
                       //if [ \$? -eq 0 ]
                       //then
                           //echo tests pass
                       //else
                           //echo tests failed
                           //exit 1
                       //fi
                    //"""
//                }
            }
        }

 
        stage('Deploy to Production') {
            when {
                branch 'master'
            }
            steps {
                  echo 'Testing.. Deploy to Production'
                script {
                    PRODUCTION_ALB_LISTENER_ARN="arn:aws:elasticloadbalancing:us-west-2:321442529690:listener/app/production-website/291d06690c5897fb/ac17ff1845610c14"
                    sh """
                    ./run-stack.sh example-webapp-production ${PRODUCTION_ALB_LISTENER_ARN}
                    """
                }
            }
        }
    }
}
