pipeline {
    agent any{
    }

    environment {
        git_url = "https://github.com/melek-khammessi/CIDevops.git"
        git_branch = "main"
        backend_imageName = "melek/backend"
        frontend_imageName = "melek/frontend"
        registryCredentials = "nexus_cred"
        nexus_registry = "localhost:1234"
        nexus_username = "admin"
        nexus_password = "root"
        backendDockerImage = ""
        frontendDockerImage = ""
    }

    stages {
        stage('Clone git repository') {
            steps {
                git branch: git_branch,
                url: git_url;
                script {
                    sh "ls -lart ./*"
                }
            }
        }

        stage('Compile maven project') {
            steps {
                script {
                    sh "cd backend; mvn compile; cd .."
                }
            }
        }


        stage('Build backend docker image') {
            steps{
                script {
                    backendDockerImage = docker.build("${backend_imageName}:${env.BUILD_NUMBER}", "backend/")
                }
            }
        }

        stage('Build frontend docker image') {
            steps{
                script {
                    frontendDockerImage = docker.build("${frontend_imageName}:${env.BUILD_NUMBER}", "frontend/")
                }
            }
        }

        stage('Upload backend docker image to Nexus') {
            steps{
                script {
                    docker.withRegistry("http://${nexus_registry}", registryCredentials ) {
                        backendDockerImage.push("${env.BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('Upload frontend docker image to Nexus') {
            steps{
                script {
                    docker.withRegistry("http://${nexus_registry}", registryCredentials ) {
                        frontendDockerImage.push("${env.BUILD_NUMBER}")
                    }
                }
            }
        }

        stage('Deploy app to environement') {
            steps{
                script {
                    sh "sed -i \"s/TAG=.*/TAG=${env.BUILD_NUMBER}/\" .env"
                    sh "sed -i \"s/NEXUS_REPOSITORY=.*/NEXUS_REPOSITORY=${nexus_registry}/\" .env"
                    sh "cat .env"
                    sh "docker login http://${nexus_registry} --username ${nexus_username} --password ${nexus_password}"
                    sh "docker-compose down"
                    sh "docker-compose up -d --build"
                }
            }
        }
    }

    // post {
    //     always {
    //         script {
    //             sh "docker rmi ${backend_imageName}:${env.BUILD_NUMBER} -f"
    //             sh "docker rmi localhost:1111/${backend_imageName}:${env.BUILD_NUMBER} -f"
    //             sh "docker rmi ${frontend_imageName}:${env.BUILD_NUMBER} -f"
    //             sh "docker rmi localhost:1111/${frontend_imageName}:${env.BUILD_NUMBER} -f"
    //         }
    //     }
    // }
}