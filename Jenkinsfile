pipeline {
    agent any
    
    tools {
        maven "maven3"
    }
    environment {
        SCANNER_HOME=tool "sonar-scanner"
    }

    stages {
        stage('Git CheckOut') {
            steps {
                git branch: 'main', url: 'https://github.com/sachi3050/Mission.git'
            }
        }
        stage('Code Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test Cases') {
            steps {
                sh 'mvn test -DskipTests=True'
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQUBE') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner  -Dsonar.projectName=Mission \
                    -Dsonar.projectKey=Mission \
                    -Dsonar.java.binaries=. '''
                }
            }
        }
        stage('Build Application') {
            steps {
                sh 'mvn package -DskipTests=True'
            }
        }
        stage('Deploy Artifacts to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'Mission-Project-Config-File', maven: 'maven3', traceability: true) {
                    sh 'mvn deploy -DskipTests=true'
                }
            }
        }
        stage('Build and Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker-Cred') {
                        sh 'docker build -t sachidananda06/mission:latest1 .'
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --format table -o trivy-image-report.html sachidananda06/mission:latest1'
            }
        }
        stage('Push Image to DockerHub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker-Cred') {
                        sh 'docker push sachidananda06/mission:latest1'
                    }
                }
            }
        }
        stage('Push Image to DockerHub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Docker-Cred') {
                        sh 'docker run -d -p 8083:8080 sachidananda06/mission:latest1'
                    }
                }
            }
        }
        stage('Deploy to K8s Cluster') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'Sachi-EKS', contextName: '', credentialsId: 'K8-Token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://9B8828DC75CE97383DC441E40AD0E732.gr7.us-east-1.eks.amazonaws.com') {
                    sh 'kubectl apply -f ds.yml'
                    sleep 45
                }
            }
        }
        stage('Verify Deployents') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'Sachi-EKS', contextName: '', credentialsId: 'K8-Token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://9B8828DC75CE97383DC441E40AD0E732.gr7.us-east-1.eks.amazonaws.com') {
                    sh 'kubectl get pods -n webapps'
                    sh 'kubectl get svc -n webapps'
                }
            }
        }
        
    }
}
