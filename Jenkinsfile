pipeline {
    agent {label 'connect-vmtest'}
    environment {

        GITLAB_IMAGE_NAME = "registry.gitlab.com/watthachai/simple-api-docker-registry"
        VMTEST_ROBOT_WORKSPACE = "/home/vmtest/workspace/simple-api-pipeline@2/simple-api-robot/"
        VMTEST_MAIN_WORKSPACE = "/home/vmtest/workspace/simple-api-pipeline@2/"
    }
    stages {
        stage('Deploy Docker Compose') {
            agent {label 'connect-vmtest'}
            steps {
                sh "docker compose up -d --build"
            }
        }
        stage("Run Tests") {
            agent {label 'connect-vmtest'}
            steps {
                sh """
                    . /home/vmtest/env/bin/activate
                    cd ${VMTEST_ROBOT_WORKSPACE}
                    robot test-calculate.robot
                    cd ${VMTEST_MAIN_WORKSPACE}
                    python3 -m unittest unit_test.py -v
                    """
            }
        }
        stage("Delivery") {
            agent {label 'connect-vmtest'}
            steps {
                withCredentials(
                    [usernamePassword(
                        credentialsId: 'gitlab-registry',
                        passwordVariable: 'gitlabPassword',
                        usernameVariable: 'gitlabUser'
                    )]
                ){
                    sh "docker login registry.gitlab.com -u ${gitlabUser} -p ${gitlabPassword}"
                    sh "docker tag ${GITLAB_IMAGE_NAME} ${GITLAB_IMAGE_NAME}:${env.BUILD_NUMBER}"
                    sh "docker push ${GITLAB_IMAGE_NAME}:${env.BUILD_NUMBER}"
                    sh "docker rmi ${GITLAB_IMAGE_NAME}:${env.BUILD_NUMBER}"
                    
                }
            }
        }
        stage("") {
            agent {label 'connect-vmtest'}
            steps {
                withCredentials(
                    [usernamePassword(
                        credentialsId: 'gitlab-registry',
                        passwordVariable: 'gitlabPassword',
                        usernameVariable: 'gitlabUser'
                    )]
                ){
                    sh "docker login registry.gitlab.com -u ${gitlabUser} -p ${gitlabPassword}"
                    sh "docker tag ${GITLAB_IMAGE_NAME} ${GITLAB_IMAGE_NAME}:${env.BUILD_NUMBER}"
                    sh "docker push ${GITLAB_IMAGE_NAME}:${env.BUILD_NUMBER}"
                    sh "docker rmi ${GITLAB_IMAGE_NAME}:${env.BUILD_NUMBER}"
                }
            }
        }
    }
}