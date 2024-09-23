pipeline {
    agent any
    triggers {
        pollSCM('H/1 * * * *')  // Check every 1 minute
    }
    environment {
        DOCKER_IMAGE = 'registry.gitlab.com/watthachai/simple-api-docker-registry:latest'
        DOCKER_DIR = '~/simple-api/app'
        API_ROBOT_DIR = '~/simple-api/simple-api-robot'
        SIMEPLE_API_DIR = '~/simple-api'
        VMTEST = "vmtest@10.211.55.5"
        VMPREPROD = "vmpreprod@10.211.55.3"
    }
    stages {
        stage('CI/CD Pipeline for VMTEST') {
            steps {
                script {
                        // Group SSH commands into a single session
                        sh """
                        ssh ${VMTEST} << 'EOF'
                        source ~/env/bin/activate
                        git clone https://github.com/SDPxMTNRWTPKKS/simple-api
                        cd ~/simple-api
                        git pull origin main
                        docker-compose -f docker-compose.yaml up -d --build
                        docker ps
                        docker images
                        python3 -m unittest unit_test.py -v
                        cd ${SIMEPLE_API_DIR}
                        cd ${API_ROBOT_DIR}
                        git set-url origin 
                        git pull origin main
                        robot test-calculate.robot
                        docker login registry.gitlab.com -u watthachai -p glpat-RTqqMg1owtgdVAZLqW99
                        docker push ${DOCKER_IMAGE}
                        << 'EOF'
                        """
                }
            }
        }
        stage('CI/CD Pipeline for VMPREPROD') {
            steps {
                script {
                        // Group SSH commands into a single session
                        sh """
                        ssh ${VMPREPROD} << 'EOF'
                        docker stop simple-api-container 
                        docker rm -f simple-api-container
                        docker login registry.gitlab.com -u watthachai -p glpat-RTqqMg1owtgdVAZLqW99
                        docker pull ${DOCKER_IMAGE}
                        docker run -d --name simple-api-container -p 5000:5000 ${DOCKER_IMAGE}
                        << 'EOF'
                        """
                    
                }
            }
        }
    }
}