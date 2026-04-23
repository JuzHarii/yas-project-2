pipeline {
    agent any

    parameters {
        choice(name: 'SERVICE_NAME', 
            choices: [
                'tax-service', 
                'cart-service', 
                'product-service', 
                'identity-service',
                'media-service', 
                'order-service', 
                'rating-service', 
                'customer-service', 
                'location-service', 
                'inventory-service', 
                'search-service',
                'payment-service', 
                'promotion-service'
            ], 
            description: 'Chọn Service bạn vừa sửa code'
        )
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

        stage('1.5. Biên dịch Code Java (Maven)') {
            steps {
                script {
                    def folderName = params.SERVICE_NAME.replace("-service", "")
                    sh """
                    docker run --rm --volumes-from jenkins-server -w \${WORKSPACE} maven:latest \
                    mvn clean package -pl ${folderName} -am -DskipTests
                    """
                }
            }
        }

        stage('2. Build Image') {
            steps {
                script {
                    env.COMMIT_ID = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    
                    sh "docker build -t juzharii/${params.SERVICE_NAME}:${env.COMMIT_ID} -f ./services/${params.SERVICE_NAME}/Dockerfile ."
                }
            }
        }

        stage('3. Push Image lên Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-id', 
                                                passwordVariable: 'DOCKER_PASSWORD', 
                                                usernameVariable: 'DOCKER_USERNAME')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin 2>/dev/null'
                    
                    sh "docker push juzharii/${params.SERVICE_NAME}:${env.COMMIT_ID}"
                }
            }
        }

        stage('4. CD - Deploy bằng Helm') {
            steps {
                script {
                    sh """
                    /usr/local/bin/helm upgrade --install yas-release ./deploy/helm/yas-env \
                    --set ${params.SERVICE_NAME}.tag=${env.COMMIT_ID} \
                    --kubeconfig /var/jenkins_home/.kube/config
                    """
                }
            }
        }
    }
}
