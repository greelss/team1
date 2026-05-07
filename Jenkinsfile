pipeline {
    agent any
    
    tools {
        jdk 'JDK21'
        maven 'M3'
    }

    environment {
        DOCKER_IMAGE_NAME = "spring-petclinic"
        DOCKERHUB_CRED = credentials('dockerCredentials')
        DOCKER_API_VERSION = '1.43'
        COMPOSE_API_VERSION = '1.43'
        // S3를 위한 변수 (필요시 추가)
        // REGION = "ap-northeast-2"
        // AWS_CREDENTIAL_NAME = "aws-credentials-id"
    }

    stages {
        stage('Git Clone') {
            steps {
                git url: 'https://github.com/greelss/spring-petclinic.git', branch: 'main'
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }

        stage('Docker Build && Push') {
            steps {
                sh '''
                docker build -t ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} .
                docker tag ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} greelss/${DOCKER_IMAGE_NAME}:latest
                echo ${DOCKERHUB_CRED_PSW} | docker login -u ${DOCKERHUB_CRED_USR} --password-stdin
                docker push greelss/${DOCKER_IMAGE_NAME}:latest
                '''
            }
            // Docker 단계가 끝나자마자 바로 삭제를 실행 (원하셨던 위치)
            post {
                always {
                    sh '''
                    docker rmi -f ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} || true
                    docker rmi -f greelss/${DOCKER_IMAGE_NAME}:latest || true
                    '''
                }
            }
        }

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
    } // 여기서 모든 stages 종료
}
