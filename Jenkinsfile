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
        ROOT_PASSWORD_VMTEST="vmtest2admin"
        ROOT_PASSWORD_VMPREPROD="vmpreprod3admin"
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
                            sudo docker-compose down
                            sudo docker-compose -f docker-compose.yaml up -d --build --remove-orphans
                            sudo docker ps
                            sudo docker images
                            python3 -m unittest unit_test.py -v
                            cd ${API_ROBOT_DIR}
                            git pull origin main
                            robot test-calculate.robot
                            echo "${DOCKER_CREDS_PSW}" | sudo docker login registry.gitlab.com -u ${DOCKER_CREDS_USR} --password-stdin
                            sudo docker push ${DOCKER_IMAGE}
                        << 'EOF'
                        """
                    }
                }
            }
        }
        stage('CI/CD Pipeline for VMPREPROD') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds-id', usernameVariable: 'DOCKER_CREDS_USR', passwordVariable: 'DOCKER_CREDS_PSW')]) {
                    script {
                        sh "ssh ${VMPREPROD} "
                        sh "sudo docker stop simple-api-container || true && sudo docker rm -f simple-api-container || true"
                        sh "echo '${DOCKER_CREDS_PSW}' | sudo docker login registry.gitlab.com -u ${DOCKER_CREDS_USR} --password-stdin && docker pull ${DOCKER_IMAGE}"
                        sh "echo '${ROOT_PASSWORD_VMPREPROD}' | sudo -S docker run -d --name simple-api-container -p 5000:5000 ${DOCKER_IMAGE} || true"
                    }
                }
            }
        }
    }
}
