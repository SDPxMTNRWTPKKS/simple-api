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
                //sh "docker build -t ${IMAGE_NAME} ."
            }
        }
        
    }
}