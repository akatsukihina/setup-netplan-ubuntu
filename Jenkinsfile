pipeline {
    agent {
        label 'agent-build'
    }

    environment {
        REGISTRY        = 'registry-lab.akatsukihina.com.vn'
        IMAGE_NAME      = 'ansible/network-setup'
        IMAGE_TAG       = "${BUILD_NUMBER}"
        GIT_REPO        = 'https://gitlab.akatsukihina.com.vn/ansible/network-setup.git'
        GIT_CREDENTIAL  = 'gitlab-auto'
        REG_CREDENTIAL  = 'registry'
    }

    stages {

        stage('Checkout source from GitLab') {
            steps {
                git branch: 'main',
                    credentialsId: "${GIT_CREDENTIAL}",
                    url: "${GIT_REPO}"
            }
        }

        stage('Login Registry') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${REG_CREDENTIAL}",
                        usernameVariable: 'REG_USER',
                        passwordVariable: 'REG_PASS'
                    )
                ]) {
                    sh '''
                        echo "$REG_PASS" | docker login ${REGISTRY} \
                          -u "$REG_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Build Image') {
            steps {
                sh '''
                    docker build \
                      -t ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} \
                      .
                '''
            }
        }

        stage('Push Image') {
            steps {
                sh '''
                    docker push ${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }
    }

    post {
        cleanup {
            sh '''
                docker logout ${REGISTRY} || true
                docker image prune -f
            '''
        }
    }
}
