pipeline {
    agent {label 'connect-vmtest'}
    environment {

        GITLAB_IMAGE_NAME = "registry.gitlab.com/watthachai/simple-api-docker-registry"
    }
    stages {
        stage('Test Node VMTEST'){
            steps {
                sh "echo ${env.APP_NAME}"
                sh "docker version"
                //sh "docker build -t ${IMAGE_NAME} ."
            }
        }
        stage('Test Node VMPREPROD'){
            agent {label 'connect-vmpreprod'}
            steps {
                sh "echo ${env.APP_NAME}"
                sh "docker version"
                sh "docker compose up -d --build"
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
                    sh "docker push ${GITLAB_IMAGE_NAME}"
                    sh "docker push ${GITLAB_IMAGE_NAME}:${env.BUILD_NUMBER}"
                    //sh "docker rmi ${GITLAB_IMAGE_NAME}"
                    //sh "docker rmi ${GITLAB_IMAGE_NAME}:${env.BUILD_NUMBER}"
                }
            }
        }
    }
}