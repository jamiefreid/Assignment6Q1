pipeline {
    agent any

    environment {
        ImageRegistry = 'jamiefreid' // Updated to my Docker username
        EC2_IP = '54.157.181.241' // Updated to my EC2 instance's IP
        DockerComposeFile = 'docker-compose.yml'
        DotEnvFile = '.env'
        // Added for local Windows execution
        SSH_KEY_PATH = 'C:\\Users\\Reppamon\\Desktop\\School\\DevOps\\Assignments\\Assignment6Question1\\Assignment6Q1.pem'
    }

    stages {

        stage("buildImage") {
            steps {
                script {
                    echo "Building Docker Image..."
                    // Swapped 'sh' for 'bat' for Windows execution
                    bat "docker build -t ${ImageRegistry}/${env.JOB_NAME}:${env.BUILD_NUMBER} ."
                }
            }
        }

        stage("pushImage") {
            steps {
                script {
                    echo "Pushing Image to DockerHub..."
                    // Updated to my specific credential ID
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        bat "echo %PASS%| docker login -u %USER% --password-stdin"
                        bat "docker push ${ImageRegistry}/${env.JOB_NAME}:${env.BUILD_NUMBER}"
                    }
                }
            }
        }

        stage("deployCompose") {
            steps {
                script {
                    echo "Deploying with Docker Compose..."
                    // Replaced sshagent with native Windows scp/ssh commands using my key path
                    bat """
                    scp -i ${SSH_KEY_PATH} -o StrictHostKeyChecking=no ${DotEnvFile} ${DockerComposeFile} ubuntu@${EC2_IP}:/home/ubuntu
                    """
                    
                    // We must export the variables in the same SSH session before running compose
                    bat """
                    ssh -i ${SSH_KEY_PATH} -o StrictHostKeyChecking=no ubuntu@${EC2_IP} "export ImageRegistry=${ImageRegistry} && export JOB_NAME=${env.JOB_NAME} && export BUILD_NUMBER=${env.BUILD_NUMBER} && docker compose -f /home/ubuntu/${DockerComposeFile} --env-file /home/ubuntu/${DotEnvFile} down"
                    """
                    
                    bat """
                    ssh -i ${SSH_KEY_PATH} -o StrictHostKeyChecking=no ubuntu@${EC2_IP} "export ImageRegistry=${ImageRegistry} && export JOB_NAME=${env.JOB_NAME} && export BUILD_NUMBER=${env.BUILD_NUMBER} && docker compose -f /home/ubuntu/${DockerComposeFile} --env-file /home/ubuntu/${DotEnvFile} up -d"
                    """
                }
            }
        }
    }
}