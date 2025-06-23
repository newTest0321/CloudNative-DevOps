pipeline {
    agent any
    
    environment {
        SONAR_HOME = tool "sonar"
        DOCKER_USERNAME = "atkaridarshan04"
    }
    
    parameters {
        string(name: 'FRONTEND_DOCKER_TAG', defaultValue: 'latest', description: 'Docker tag for frontend image')
        string(name: 'BACKEND_DOCKER_TAG', defaultValue: 'latest', description: 'Docker tag for backend image')
    }
    
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
                echo "Workspace cleaned"
            }
        }
        
        stage('Checkout Code') {
            steps {
                git url: "https://github.com/atkaridarshan04/CloudNative-DevOps-Blueprint.git", branch: "main"
                echo "Code checked out from GitHub"
            }
        }
        
        stage('Trivy File Scan') {
            steps {
                sh "trivy fs --format table -o trivy-report.html ."
                echo "Trivy filesystem scan completed"
            }
        }
        
        // stage('OWASP Dependency Check') {
        //     steps {
        //         dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'OWASP'
        //         dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        //         echo "OWASP dependency check completed"
        //     }
        //     post {
        //         failure {
        //             echo "OWASP scan failed but continuing pipeline"
        //             script {
        //                 currentBuild.result = 'UNSTABLE'
        //             }
        //         }
        //     }
        // }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("sonar") {
                    sh """
                        ${SONAR_HOME}/bin/sonar-scanner \
                        -Dsonar.projectName=bookstore \
                        -Dsonar.projectKey=bookstore \
                        -Dsonar.sources=src
                    """
                }
                echo "SonarQube analysis completed"
            }
        }
        
        stage('Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
                echo "Quality gate check completed"
            }
        }
        
        stage('Build Frontend Image') {
            steps {
                dir('src/frontend') {
                    sh """
                        echo "Building frontend Docker image..."
                        docker build -t ${DOCKER_USERNAME}/bookstore-frontend:${params.FRONTEND_DOCKER_TAG} .
                    """
                }
                echo "Frontend image built successfully"
            }
        }
        
        stage('Build Backend Image') {
            steps {
                dir('src/backend') {
                    sh """
                        echo "Building backend Docker image..."
                        docker build -t ${DOCKER_USERNAME}/bookstore-backend:${params.BACKEND_DOCKER_TAG} .
                    """
                }
                echo "Backend image built successfully"
            }
        }
        
        stage('Scan Docker Images') {
            steps {
                sh """
                    echo "Scanning Docker images for vulnerabilities..."
                    trivy image --format table ${DOCKER_USERNAME}/bookstore-frontend:${params.FRONTEND_DOCKER_TAG} || true
                    trivy image --format table ${DOCKER_USERNAME}/bookstore-backend:${params.BACKEND_DOCKER_TAG} || true
                """
                echo "Docker image scanning completed"
            }
        }
        
        stage('Push Images to DockerHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-token', 
                    passwordVariable: 'DOCKER_PASSWORD', 
                    usernameVariable: 'DOCKER_USER'
                )]) {
                    sh """
                        echo "Logging into DockerHub..."
                        docker login -u ${DOCKER_USER} -p ${DOCKER_PASSWORD}
                        
                        echo "Pushing frontend image..."
                        docker push ${DOCKER_USERNAME}/bookstore-frontend:${params.FRONTEND_DOCKER_TAG}
                        
                        echo "Pushing backend image..."
                        docker push ${DOCKER_USERNAME}/bookstore-backend:${params.BACKEND_DOCKER_TAG}
                    """
                }
                echo "Images pushed to DockerHub successfully"
            }
        }
    }
    
    post {
        always {
            echo "Pipeline execution completed"
            archiveArtifacts artifacts: '*.html,**/*.xml', allowEmptyArchive: true
        }
        
        success {
            echo "✅ Pipeline completed successfully!"
            build job: "bookstore-cd", 
                  parameters: [
                      string(name: 'FRONTEND_DOCKER_TAG', value: "${params.FRONTEND_DOCKER_TAG}"),
                      string(name: 'BACKEND_DOCKER_TAG', value: "${params.BACKEND_DOCKER_TAG}")
                  ],
                  wait: false
        }
        
        failure {
            echo "❌ Pipeline failed!"
        }
        
        unstable {
            echo "⚠️ Pipeline completed with warnings"
        }
    }
}
