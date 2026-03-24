pipeline {
    agent any

    environment {
        EC2_USER = "ubuntu"
        EC2_HOST = "3.38.108.207"
        EC2_KEY  = "/var/jenkins_home/.ssh/id_rsa"

        APP_DIR  = "/home/ubuntu/app"
        APP_NAME = "demo1-0.0.1-SNAPSHOT.jar"
        APP_PORT = "9090"
    }

    stages {

        stage('Git Checkout') {
            steps {
                git url: 'https://github.com/vlria-os/myapp.git', branch: 'main'
            }
        }

        stage('Build') {
            steps {
                sh '''
chmod +x ./gradlew
./gradlew clean bootJar -x test
'''
            }
        }

        stage('Deploy & Run on EC2') {
            steps {
                sh """
echo " Deploy start"

# 1. JAR 파일 EC2로 전송
scp -i ${EC2_KEY} -o StrictHostKeyChecking=no \
build/libs/${APP_NAME} \
${EC2_USER}@${EC2_HOST}:${APP_DIR}/${APP_NAME}

# 2. EC2에서 기존 프로세스 종료 후 재실행 ( EOF와 EOF 사이에는 공백이 있으면 안되므로 임의의 들여쓰기 X)
ssh -i ${EC2_KEY} -o StrictHostKeyChecking=no ${EC2_USER}@${EC2_HOST} << EOF
pkill -f ${APP_NAME} || true
echo "Starting new version"

nohup java -jar ${APP_DIR}/${APP_NAME} \\
  --server.address=0.0.0.0 \\
  --server.port=${APP_PORT} \\
  > ${APP_DIR}/app.log 2>&1 &
EOF
"""
            }
        }
    }

    post {
        success {
            echo " Deployment completed: http://${EC2_HOST}:${APP_PORT}"
        }
        failure {
            echo " Deployment failed"
        }
    }
}
