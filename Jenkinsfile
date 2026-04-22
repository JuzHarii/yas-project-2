pipeline {
    agent any

    parameters {
        choice(name: 'SERVICE_NAME', choices: ['tax-service', 'cart-service', 'product-service', 'identity-service'], description: 'Chọn Service bạn vừa sửa code')
        string(name: 'BRANCH_NAME', defaultValue: 'main', description: 'Tên nhánh (branch) bạn muốn test (VD: dev_tax_service)')
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-id')
        DOCKER_USER = "juzharii" 
    }

    stages {
        stage('1. Kéo Code (Checkout)') {
            steps {
                checkout scm
            }
        }

        stage('2. Build Docker Image') {
            steps {
                script {
                    def folderName = params.SERVICE_NAME.replace("-service", "")
                    sh "docker build -t ${DOCKER_USER}/${params.SERVICE_NAME}:${params.BRANCH_NAME} -f ./${folderName}/Dockerfile ."
                }
            }
        }

        stage('3. Push Image lên Docker Hub') {
            steps {
                script {
                    sh "echo \$DOCKERHUB_CREDENTIALS_PSW | docker login -u \$DOCKERHUB_CREDENTIALS_USR --password-stdin"
                    sh "docker push ${DOCKER_USER}/${params.SERVICE_NAME}:${params.BRANCH_NAME}"
                }
            }
        }

        stage('4. CD - Deploy bằng Helm') {
            steps {
                script {
                    sh """
                    helm upgrade --install yas-release ./deploy/helm/yas-env \
                    --set ${params.SERVICE_NAME}.tag=${params.BRANCH_NAME}
                    """
                }
            }
        }
    }
}