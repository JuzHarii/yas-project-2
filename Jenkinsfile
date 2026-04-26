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
                    def folderName = params.SERVICE_NAME.replace("-service", "")
                    
                    
                    sh "docker build -t juzharii/${params.SERVICE_NAME}:${env.COMMIT_ID} -f ./${folderName}/Dockerfile ./${folderName}"
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

        stage('4. CD - ArgoCD lo phần Deploy') {
            steps {
                script {
                    // Phân luồng môi trường dựa vào nhánh
                    def envFile = "values-dev.yaml" // Mặc định đẩy vào Dev
                    if (params.BRANCH_NAME == "main" || params.BRANCH_NAME.startsWith("release")) {
                        envFile = "values-staging.yaml" // Đẩy vào Staging nếu là nhánh main/release
                    }

                    echo "Đang cập nhật Tag ${env.COMMIT_ID} cho ${params.SERVICE_NAME} vào file ${envFile}"

                    // Đồng bộ tên tax-service -> tax_service 
                    def yamlKey = params.SERVICE_NAME.replace("-", "_")

                    // Tìm đúng khối của service và thay thế dòng tag bên dưới nó
                    sh """
                    sed -i '/^${yamlKey}:/,/tag:/ s/tag: .*/tag: "${env.COMMIT_ID}"/' ./deploy/helm/yas-env/${envFile}
                    """

                    // Commit và Push lên GitHub
                    withCredentials([usernamePassword(credentialsId: 'github-credentials', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                        sh """
                        git config user.email "jenkins-bot@hcmus.edu.vn"
                        git config user.name "Jenkins GitOps Bot"
                        git add ./deploy/helm/yas-env/${envFile}
                        git commit -m "Auto-update ${params.SERVICE_NAME} to version ${env.COMMIT_ID} in ${envFile}"
                        
                        git push https://${GIT_USERNAME}:${GIT_PASSWORD}@github.com/JuzHarii/yas-project-2.git HEAD:${params.BRANCH_NAME}
                        """
                    }
                }
            }
        }
    }
}
