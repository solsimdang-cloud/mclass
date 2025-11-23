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
        stages('Git Checkout'){
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
    }

}