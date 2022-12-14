pipeline {
    agent any
    environment {
        //be sure to replace "bhavukm" with your own Docker Hub username
        DOCKER_IMAGE_NAME = "varunk777/train-schedule"
        PROJECT_ID = 'intrepid-fiber-355806'
        CLUSTER_NAME = 'edureka'
        LOCATION = 'us-central1-c'
        CREDENTIALS_ID = 'kubernetes'
    }
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            
            steps {
                script {
                    app = docker.build(DOCKER_IMAGE_NAME)
                    app.inside {
                        sh 'echo Hello, World!'
                    }
                }
            }
        }
        stage('Push Docker Image') {
            
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'varunk777') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage('CanaryDeploy') {
            
            environment { 
                CANARY_REPLICAS = 1
            }
            steps {
                step([$class: 'KubernetesEngineBuilder', 
                        projectId: env.PROJECT_ID,
                        clusterName: env.CLUSTER_NAME,
                        zone: env.LOCATION,
                        manifestPattern: 'train-schedule-kube-canary.yml',
                        credentialsId: env.CREDENTIALS_ID,
                        verifyDeployments: true])
                
            }
        }
        stage('DeployToProduction') {
            
            environment { 
                CANARY_REPLICAS = 0
            }
            steps {
                step([$class: 'KubernetesEngineBuilder', 
                        projectId: env.PROJECT_ID,
                        clusterName: env.CLUSTER_NAME,
                        zone: env.LOCATION,
                        manifestPattern: 'train-schedule-kube.yml',
                        credentialsId: env.CREDENTIALS_ID,
                        verifyDeployments: true])
            }
        }
    }
}
