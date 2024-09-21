pipeline {
    agent any
    triggers {
        pollSCM('H/1 * * * *')  // Check every 1 minute
    }
    environment {
        DOCKER_IMAGE = 'registry.gitlab.com/watthachai/simple-api-docker-registry:latest'
        DOCKER_DIR = '~/simple-api/app'
        API_ROBOT_DIR = '~/simple-api/simple-api-robot'
        SIMPLE_API_DIR = '~/simple-api'  // Fixed typo
        VMTEST = "vmtest@10.211.55.8"
        VMPREPROD = "vmpreprod@10.211.55.9"
        DOCKER_CREDS = credentials('docker-creds-id')  // Use Jenkins credentials
    }
    stages {
        stage('CI/CD Pipeline for VMTEST') {
            steps {
                script {
                    sh """
                    ssh ${VMTEST} << 'E0F'
                        source ~/env/bin/activate
                        if [ ! -d "~/simple-api" ]; then
                        git clone https://github.com/SDPxMTNRWTPKKS/simple-api
                        fi
                        cd ~/simple-api
                        git pull origin main
                        sudo docker-compose down
                        sudo docker-compose -f docker-compose.yaml up -d --build --remove-orphans
                        sudo docker ps
                        sudo docker images
                        python3 -m unittest unit_test.py -v
                        cd ${SIMPLE_API_DIR}
                        cd ${API_ROBOT_DIR}
                        git pull origin main
                        robot test-calculate.robot
                        sudo docker login registry.gitlab.com -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}
                        sudo docker push ${DOCKER_IMAGE}
                    << 'E0F'
                    """
                }
            }
        }
        stage('CI/CD Pipeline for VMPREPROD') {
            steps {
                script {
                    sh """
                    ssh ${VMPREPROD} << 'E0F'
                        sudo docker stop simple-api-container || true
                        sudo docker rm -f simple-api-container || true
                        sudo docker login registry.gitlab.com -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}
                        sudo docker pull ${DOCKER_IMAGE}
                        sudo docker run -d --name simple-api-container -p 5000:5000 ${DOCKER_IMAGE}
                    << 'E0F'
                    """
                }
            }
        }
    }
}
