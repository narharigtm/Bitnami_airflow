pipeline {
    agent any
    environment {
        // Define the EC2 instance details
        EC2_USER = 'ubuntu'  // Username for SSH (change to your EC2's username)
        EC2_HOST = '35.89.164.55'  // EC2 public IP or DNS
        EC2_KEY = credentials('ssh_key')  // Jenkins SSH key credential ID
    }
    stages {
        stage('Git Checkout') {
            steps {
                script {
                    git branch: 'main',
                        credentialsId: 'git_credentials',
                        url: 'https://github.com/narharigtm/Bitnami_airflow.git'
                    sh 'ls'
                }
            }
        }
        stage('Build the Image') {
            steps {
                script {
                    echo 'Building Docker images...'
                    sh 'cd postgresql && sudo docker build -t postgresqlimg:airflow .'
                    sh 'cd redis && sudo docker build -t redisimg:airflow .'
                    sh 'cd airflow && sudo docker build -t airflowimg:airflow .'
                    sh 'cd schedular && sudo docker build -t schedularimg:airflow .'
                    sh 'cd worker && sudo docker build -t workerimg:airflow .'
                    sh 'sudo docker images'
                }
            }
        }
        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker_hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo "$DOCKER_PASSWORD" | sudo docker login -u "$DOCKER_USERNAME" --password-stdin'
                }
                sh 'sudo docker image tag postgresqlimg:airflow narhari/postgresqlimg:v1'
                sh 'sudo docker image tag redisimg:airflow narhari/redisimg:v1'
                sh 'sudo docker image tag airflowimg:airflow narhari/airflowimg:v1'
                sh 'sudo docker image tag schedularimg:airflow narhari/schedularimg:v1'
                sh 'sudo docker image tag workerimg:airflow narhari/workerimg:v1'
                sh 'sudo docker push narhari/postgresqlimg:v1'
                sh 'sudo docker push narhari/redisimg:v1'
                sh 'sudo docker push narhari/airflowimg:v1'
                sh 'sudo docker push narhari/schedularimg:v1'
                sh 'sudo docker push narhari/workerimg:v1'
            }
        }
        stage('Pull and Run Containers on EC2') {
            steps {
                script {
                    // SSH into EC2 instance, pull Docker images, and run containers
                    sh '''#!/bin/bash
                        ssh -o StrictHostKeyChecking=no -i ${EC2_KEY} ${EC2_USER}@${EC2_HOST} "
                            echo 'Pulling Docker images...'
                            sudo docker pull narhari/postgresqlimg:v1
                            sudo docker pull narhari/redisimg:v1
                            sudo docker pull narhari/airflowimg:v1
                            sudo docker pull narhari/schedularimg:v1
                            sudo docker pull narhari/workerimg:v1

                            echo 'Running Docker containers...'
                            # Create a custom network for the containers to communicate
                            sudo docker network create airflow_network

                            # Run PostgreSQL container and connect it to the custom network
                            sudo docker run -d -p 5432:5432 --name postgresql_con --network airflow_network narhari/postgresqlimg:v1

                            # Run Redis container and connect it to the custom network
                            sudo docker run -d -p 6397:6397 --name redis_con --network airflow_network narhari/redisimg:v1

                            # Run Airflow container and connect it to the custom network
                            sudo docker run -d -p 8080:8080 --name airflow_con --network airflow_network narhari/airflowimg:v1

                            # Run Scheduler container and connect it to the custom network
                            sudo docker run -d --name schedular_con --network airflow_network narhari/schedularimg:v1

                            # Run Worker container and connect it to the custom network
                            sudo docker run -d --name worker_con --network airflow_network narhari/workerimg:v1

                            
                            

                            echo 'Verifying running containers...'
                            sudo docker ps  # Verify the containers are running
                        "
                    '''
                }
            }
        }
    }
}
