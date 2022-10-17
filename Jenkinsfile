// def mainDir="Chapter02/2-jenkins-docker"
def mainDir="./"

// 아래 주소에 실제로 접속하면, docker-credential-ecr-login-helper 실행 용도의 바이너리 파일이 존재함
// https://amazon-ecr-credential-helper-releases.s3.us-east-2.amazonaws.com/0.4.0/linux-amd64/docker-credential-ecr-login
def ecrLoginHelper="docker-credential-ecr-login"
def region="ap-northeast-2"
def ecrUrl="469041045135.dkr.ecr.ap-northeast-2.amazonaws.com"
def repository="test-jenkins"// ecr 리포지터리 명
def deployHost="172.31.7.183"

pipeline {
    agent any

    stages {
        stage('Pull Codes from Github'){
            steps{
                checkout scm
            }
        }
        stage('chmod +x gradlew'){
            steps{
                sh """
                chmod +x gradlew
                """
            }
        }
        stage('Build Codes by Gradle') {
            steps {
                sh """
                cd ${mainDir}
                ./gradlew clean build
                """
            }
        }
        stage('Build Docker Image by Jib & Push to AWS ECR Repository') {
            steps {
                withAWS(region:"${region}", credentials:"ec2-temp-study-docker-kube") {
                    ecrLogin()
                    // currentBuild.number : 현재 젠킨스 빌드 번호
                    // -Djib.console : jib 관련 실행 기록들을 콘솔에 출력하기 위한 옵션
                    sh """
                        curl -O https://amazon-ecr-credential-helper-releases.s3.us-east-2.amazonaws.com/0.4.0/linux-amd64/${ecrLoginHelper}
                        chmod +x ${ecrLoginHelper}
                        mv ${ecrLoginHelper} /usr/local/bin/
                        cd ${mainDir}
                        ./gradlew jib -Djib.to.image=${ecrUrl}/${repository}:${currentBuild.number} -Djib.console='plain'
                    """
                }
            }
        }
        stage('Deploy to AWS EC2 VM'){
            steps{
            // 태그 명은 ${ecrUrl}/${repository}:${currentBuild.number} 로 지정해줬다.
                sshagent(credentials : ["deploy-soon.good-docker-msa-study"]) {
                    sh "ssh -o StrictHostKeyChecking=no ubuntu@${deployHost} \
                     'aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${ecrUrl}/${repository}; \
                      docker run -d -p 80:8080 -t ${ecrUrl}/${repository}:${currentBuild.number};'"
                }
            }
        }
    }
}