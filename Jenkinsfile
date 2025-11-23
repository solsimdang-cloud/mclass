pipeline{
    agent any //어떤 실행 서버에서든 실행가능 함

    tools {
       maven 'maven 3.9.11' //jenkins에 등록된 maven 3.9.11을 사용
    }

    environment {
        //배포에 필요한 변수 설정
        DOCKER_IMAGE = "domo-app"// 도커 이미지 이름
        CONTAINER_NAME= "springboot-container" // 도커 컨테이너 이름
        JAR_FILE_NAME = "app.jar" //복사할 jar 파일
        PORT = '8081' //컨테이너와 연결한 포트

        REMOTE_USER= "ec2-user" //원격(spring) 서버
        REMOTE_HOST= "3.34.5.111" //원격(spring) 서버 IP(public ip)

        REMOTE_DIR= "/home/ec2-user/deploy" //원격 서버에 파일 복사할 경로

        SSH_CREDENTIALS_ID = "46a26a91-1933-45ec-beff-1128b611b1ea" //JENKINS SSH자격 증명 ID

    }
    //단계별로 실행될 코드 작성
    stages{
        stage('Git Checkout'){
            steps {//실행될 실제 명령어 작성
                //jenkins가 연결된 Git 저장소에서 최신 코드 체크아웃
                //SCM(Source Code Management)
                checkout scm
            }
        }

        stage("Maven Build"){
            steps {// es 여기 젠킨스에서 메이븐 실행하는 리눅스 명렁어(sh 쉘 명령어) 작성
                //테스트는 건너뛰고 maven 빌드
                sh 'mvn clean package -DskipTests'

            }
        }


        stage('Prepare Jar'){
            steps {
                // 빌드 결과물 JAR 파일을 지정한 이름(app.jar)으로 복사
                sh 'cp target/demo-0.0.1-SNAPSHOT.jar ${JAR_FILE_NAME}'
            }
        }

        stage('Copy to Remote Server'){
            //파일을 복사하여 옮김
            steps {
                //jenkins가 원격 서버에 ssh 접속할수있도록 sshagent사용
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]){
                //원격 서버에 배포 디렉토리 생성(없으면 새로 만듦)
                sh "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_HOST} \"mkdir -p ${REMOTE_DIR}\""
                // JAR 파일과 DOCKERFILE을 원격 서버에 복사(젠킨스서버에서 만들었던 jar파일과 도커를 스프링서버에 복사해 이동함)
                sh "scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${JAR_FILE_NAME} Dockerfile ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/"
                }
            }
        }
        stage('Remote Docker Build & Deploy'){
            steps {
                sshagent(credentials: [env.SSH_CREDENTIALS_ID]){
                    sh """
ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_HOST} << ENDSSH
    cd ${REMOTE_DIR} || exit 1
    docker rm -f ${CONTAINER_NAME} || true
    docker build -t ${DOCKER_IMAGE} .
    docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${PORT} ${DOCKER_IMAGE}
ENDSSH
                    """
                }
            }
            
        }
    }
    
}