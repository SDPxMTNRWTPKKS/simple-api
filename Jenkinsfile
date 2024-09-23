pipeline {
    agent any
    triggers {
        pollSCM('H/1 * * * *')  // Check every 1 minute
    }
    environment {
        GITLAB_USER = 'watthachai'
        GITLAB_IMAGE = 'simple-api-docker-registry'
        DOCKER_IMAGE_NAME = 'simple-api-docker'
        DOCKER_IMAGE_TAG = 'latest'
        API_ROBOT_DIR = '~/simple-api/simple-api-robot'
        VMTEST = "vmtest@10.211.55.5"
        VMPREPROD = "vmpreprod@10.211.55.3"
    }
    stages {
        stage('CI/CD Pipeline for VMTEST') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'gitlabID', usernameVariable: 'DOCKER_CREDS_USR', passwordVariable: 'DOCKER_CREDS_PSW')]) {
                    script {
                        sh """
                        ssh ${VMTEST} << 'EOF'
                            source ~/env/bin/activate
                            if [ ! -d "~/simple-api" ]; then
                                git clone https://github.com/SDPxMTNRWTPKKS/simple-api
                            fi
                            cd ~/simple-api
                            git pull origin main
                            docker-compose down
                            docker-compose up -d --build --remove-orphans
                            python3 -m unittest unit_test.py -v
                            cd ${API_ROBOT_DIR}
                            robot test-calculate.robot
                            echo '${DOCKER_CREDS_PSW}' | docker login registry.gitlab.com -u '${DOCKER_CREDS_USR}' -p '${DOCKER_CREDS_PSW}'
                            docker push ${DOCKER_IMAGE}
                        << 'EOF'
                        """
                    }
                }
            }
        }
        stage('CI/CD Pipeline for VMPREPROD') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'gitlabID', usernameVariable: 'DOCKER_CREDS_USR', passwordVariable: 'DOCKER_CREDS_PSW')]) {
                    script {
                        sh """
                        ssh ${VMPREPROD} << 'EOF'
                            docker ps -a -q -f name=${DOCKER_IMAGE_NAME} | xargs -r docker rm -f"
                            docker login registry.gitlab.com -u '${DOCKER_CREDS_USR}' -p '${DOCKER_CREDS_PSW}'
                            docker pull registry.gitlab.com/watthachai/simple-api-docker-registry:${DOCKER_IMAGE_TAG}"
                            docker run -d --name ${DOCKER_IMAGE_NAME} -p 8080:5050 registry.gitlab.com/${GITLAB_USER}/${GITLAB_IMAGE}:${DOCKER_IMAGE_TAG}"
                        << 'EOF'
                        """
                    }
                }
            }
        }
    }
}
