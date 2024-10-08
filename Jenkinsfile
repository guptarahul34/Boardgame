pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'maven3'
    }

    environment {
        DOCKER_HUB=credentials('dockerhub')
        SCANNER_HOME= tool 'sonar-scanner'
    }


    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/guptarahul34/Boardgame.git'
            }
        }

        stage('Code Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Test') {
            steps {
                sh "mvn test"
            }
        }

        stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }

        stage('SonarQube Analsyis') {
            steps {
                sh "echo $SCANNER_HOME"
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=BoardGame -Dsonar.projectKey=BoardGame \
                            -Dsonar.java.binaries=. '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                  waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            }
        }

        stage('Build') {
            steps {
               sh "mvn package"
            }
        }

        // stage('Publish To Nexus') {
        //     steps {
        //        withMaven(globalMavenSettingsConfig: 'global-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
        //             sh "mvn deploy"
        //         }
        //     }
        // }

        stage('Build Docker Image & Tag') {
            steps {
               sh "docker build -t rahulgupta9794/boardgame:latest ."
            }
        }

        stage('Push Tagged Image') {
            steps {
               // withDockerRegistry(credentialsId: 'dockerhub-cred', url: 'https://hub.docker.com/repositories/rahulgupta9794') {
                sh "echo $DOCKER_HUB_PSW | docker login -u $DOCKER_HUB_USR --password-stdin "  
                sh "docker push rahulgupta9794/boardgame:latest"
                // }
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: '', credentialsId: 'k8-cred', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://172.31.8.146:6443') {
                        sh "kubectl apply -f deployment-service.yaml"
                }
            }
        }


    }
}
