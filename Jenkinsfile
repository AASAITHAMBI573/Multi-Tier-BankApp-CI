pipeline {
    agent any

    parameters{
        string(name: 'DOCKER_TAG', defaultValue: 'latest', description: 'Docker Tag')
    }

    tools {
        maven 'maven3'
    }

    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Git Checkout') {
            steps {
                git 'https://github.com/AASAITHAMBI573/Multi-Tier-BankApp-CI.git'
            }
        }

        stage('Maven Compile') {
            steps {
               sh "mvn compile"
            }
        }

        stage('Maven Test') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."
            }
        }

        stage('SonarQube Code Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=bankapp -Dsonar.projectName=bankapp -Dsonar.java.binaries=target"
                }
            }
        }
        
        
        stage('Maven Build') {
            steps {
                sh "mvn package"
            }
        }

        stage('Publish Artifact To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-settings', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh "mvn deploy"
                }
            }
        }

        stage('Docker build & Tags') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker build -t aasaithambi5/bankapp:${params.DOCKER_TAG} ."
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image.html aasaithambi5/bankapp:${params.DOCKER_TAG}"
            }
        }

        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push aasaithambi5/bankapp:${params.DOCKER_TAG}"
                    }
                }
            }
        }
        
        stage('Update YAML Manifest in Other Repo') {
            steps {
                script {
                    withCredentials([gitUsernamePassword(credentialsId: 'git-cred', gitToolName: 'Default')]) {
                    sh """
                    # Clone the repo containing the Kubernetes deployment manifest
                    git clone https://github.com/AASAITHAMBI573/Multi-Tier-BankApp-CD.git
                    cd Multi-Tier-BankApp-CD 

                    # Check if the bankapp-deployment-service.yml file exists
                    ls -l bankapp


                    # Update the Docker image tag in the YAML file
                    sed -i 's|image: aasaithambi5/bankapp:.*|image: aasaithambi5/bankapp:'${DOCKER_TAG}'|' bankapp/bankapp-deployment-service.yml

                    # Confirm the change in the updated YAML file
                    echo "Updated YAML file contents:"
                    cat bankapp/bankapp-deployment-service.yml
                    
                    # Configure Git for the commit
                    git config user.email "aasaiXXXX@gmail.com"
                    git config user.name "AASAITHAMBIXXX"

                    # Commit and push the changes back to the repository
                    git add bankapp/bankapp-deployment-service.yml
                    git commit -m "Update image tag to ${DOCKER_TAG}"
                    git push origin main
                """
                    }
                }
            }
        }
    }
}
