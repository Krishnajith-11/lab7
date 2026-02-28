pipeline {
    agent any

    environment {
        IMAGE_NAME = "kj3748/lab6-model:latest"
        CONTAINER_NAME = "2022bcs0037-test-container"
        PORT = "8000"
    }

    stages {

        stage('Pull Image') {
            steps {
                sh 'docker pull $IMAGE_NAME'
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                docker rm -f $CONTAINER_NAME || true
                docker run -d -p $PORT:$PORT --name $CONTAINER_NAME $IMAGE_NAME
                '''
            }
        }

        stage('Wait for API') {
            steps {
                sh '''
                timeout=30
                until curl -s -o /dev/null -w "%{http_code}" http://host.docker.internal:$PORT/health | grep -q 200; do
                    if [ $timeout -le- 0 ]; then
                        echo "API did not start"
                        exit 1
                    fi
                    sleep 2
                    timeout=$((timeout-2))
                done
                echo "API is ready"
                '''
            }
        }

        stage('Valid Request') {
            steps {
                sh '''
                response=$(curl -s -X POST http://host.docker.internal:$PORT/predict \
                -H "Content-Type: application/json" \
                -d @valid.json)

                echo "Valid Response: $response"
                '''
            }
        }

        stage('Invalid Request') {
            steps {
                sh '''
                status=$(curl -s -o /dev/null -w "%{http_code}" \
                -X POST http://host.docker.internal:$PORT/predict \
                -H "Content-Type: application/json" \
                -d @invalid.json)

                if [ "$status" -eq 200 ]; then
                    echo "Invalid input should not return 200"
                    exit 1
                fi
                '''
            }
        }

        stage('Stop Container') {
            steps {
                 sh '''
                docker stop $CONTAINER_NAME
                docker rm $CONTAINER_NAME
                '''
            }
        }
    }

    post {
        always {
            sh 'docker rm -f $CONTAINER || true'
        }
    }
}