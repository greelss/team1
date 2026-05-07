pipeline {
    agent any
    
    tools {
        jdk 'JDK21'
        maven 'M3'
    }

    environment {
        DOCKER_IMAGE_NAME = "spring-petclinic"
        DOCKERHUB_CRED = credentials('hshs99')
        DOCKER_API_VERSION = '1.43'
        COMPOSE_API_VERSION = '1.43'
         // REGION = "ap-northeast-2"
        // AWS_CREDENTIAL_NAME = "aws-credentials-id"
    }

    stages {
        stage('Git Clone') {
            steps {
                git url: 'https://github.com/sjh4616/spring-petclinic.git', branch: 'main'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build && Push') {
            steps {
                sh """
                    docker build -t hshs99/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} .
                    docker tag hshs99/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} hshs99/${DOCKER_IMAGE_NAME}:latest
                    echo ${DOCKERHUB_CRED_PSW} | docker login -u ${DOCKERHUB_CRED_USR} --password-stdin
                    docker push hshs99/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}
                    docker push hshs99/${DOCKER_IMAGE_NAME}:latest
                """
            }
            post {
                always {
                    sh """
                        docker rmi -f hshs99/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} || true
                        docker rmi -f hshs99/${DOCKER_IMAGE_NAME}:latest || true
                    """
                }
            }
        }

        stage('K8s Deploy') {
            steps {
                sshPublisher(publishers: [
                    sshPublisherDesc(configName: 'target', transfers: [
                        sshTransfer(execCommand: """
                            export KUBECONFIG=/home/k8s-master/.kube/config
                            kubectl set image deployment/team1-deployment spring-petclinic=hshs99/spring-petclinic:v${env.BUILD_NUMBER}
                            kubectl rollout status deployment/team1-deployment
                        """)
                    ])
                ])
            }
        }
    } // stages 종료
} // pipeline 종료
        // stage('Upload s3') {
        //     steps {
        //         echo "Upload to S3"
        //         dir("${env.WORKSPACE}") {
        //             sh 'zip -r scripts.zip ./scripts appspec.yml'
        //             withAWS(region:"${REGION}", credentials: "${AWS_CREDENTIAL_NAME}"){
        //                 s3Upload(file:"scripts.zip", bucket:"aws02-codedeploy-bucket")
        //             }
        //             sh 'rm -rf ./scripts.zip'
        //         }
        //     }
        // }
