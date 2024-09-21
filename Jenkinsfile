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
    }
    stages {
        stage('CI/CD Pipeline for VMTEST') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds-id', usernameVariable: 'DOCKER_CREDS_USR', passwordVariable: 'DOCKER_CREDS_PSW')]) {
                    script {
                        sh """
                        ssh ${VMTEST} << 'EOF'
                            source ~/env/bin/activate
                            if [ ! -d "~/simple-api" ]; then
                                git clone https://github.com/SDPxMTNRWTPKKS/simple-api
                            fi
                            cd ~/simple-api
                            git pull origin main
                            echo ${DOCKER_CREDS_PSW} | sudo -S docker-compose down
                            echo ${DOCKER_CREDS_PSW} | sudo -S docker-compose -f docker-compose.yaml up -d --build --remove-orphans
                            echo ${DOCKER_CREDS_PSW} | sudo -S docker ps
                            echo ${DOCKER_CREDS_PSW} | sudo -S docker images
                            python3 -m unittest unit_test.py -v
                            cd ${SIMPLE_API_DIR}
                            cd ${API_ROBOT_DIR}
                            git pull origin main
                            robot test-calculate.robot
                            echo ${DOCKER_CREDS_PSW} | sudo -S docker login registry.gitlab.com -u ${DOCKER_CREDS_USR} --password-stdin
                            echo ${DOCKER_CREDS_PSW} | sudo -S docker push ${DOCKER_IMAGE}
                        << EOF
                        """
                    }
                }
            }
        }
        stage('CI/CD Pipeline for VMPREPROD') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds-id', usernameVariable: 'DOCKER_CREDS_USR', passwordVariable: 'DOCKER_CREDS_PSW')]) {
                    script {
                        sh """
                        ssh ${VMPREPROD} << 'EOF'
                            echo ${DOCKER_CREDS_PSW} | sudo -S docker stop simple-api-container || true
                            echo ${DOCKER_CREDS_PSW} | sudo -S docker rm -f simple-api-container || true
                            echo ${DOCKER_CREDS_PSW} | sudo -S docker login registry.gitlab.com -u ${DOCKER_CREDS_USR} --password-stdin
                            echo ${DOCKER_CREDS_PSW} | sudo -S docker pull ${DOCKER_IMAGE}
                            echo ${DOCKER_CREDS_PSW} | sudo -S docker run -d --name simple-api-container -p 5000:5000 ${DOCKER_IMAGE}
                        << EOF
                        """
                    }
                }
            }
        }
    }
}