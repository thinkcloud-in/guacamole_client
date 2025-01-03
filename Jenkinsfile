/*
 * Licensed to the Apache Software Foundation (ASF) under one or more
 * contributor license agreements. See the NOTICE file distributed with
 * this work for additional information regarding copyright ownership.
 * The ASF licenses this file to You under the Apache License, Version 2.0
 * (the "License"); you may not use this file except in compliance with
 * the License. You may obtain a copy of the License at
 *
 * http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
// =======
pipeline {
    agent any

    environment {
        TARGET_SERVER = '172.16.0.103'
        TARGET_USER = 'root'
        TARGET_PASSWORD = 'Teamw0rk@1'
        DEPLOY_PATH = 'Horizan_reports'
        REPO_URL = 'https://github.com/thinkcloud-in/guacamole_client.git'
        IMAGE_NAME = 'arpits2931/gucamole_openid:1.1.1'
        WORK_DIR = 'temp_guacamole_build'
        DOCKER_CREDENTIALS_ID = 'docker-id'
    }

    stages {
        stage('Git Checkout and Build Image') {
            steps {
                script {
                    sh """
                        sshpass -p '${TARGET_PASSWORD}' ssh -T -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_SERVER} '
                        
                        # Create a temporary working directory
                        mkdir -p ${WORK_DIR}
                        
                        # Change to the directory
                        cd ${WORK_DIR}
                       
                        # Clone the repository
                        git clone --single-branch --branch main ${REPO_URL} .
                        cd guacamole_client
                        # Build the Docker image
                        docker build -t ${IMAGE_NAME} .
                        
                        # Cleanup: Remove the temporary directory
                        cd ..
                        rm -rf ${WORK_DIR}
                        '
                    """
                }
            }
        }

        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry([credentialsId: DOCKER_CREDENTIALS_ID]) {
                        sh "docker build -t ${IMAGE_NAME} ."
                    }
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry([credentialsId: DOCKER_CREDENTIALS_ID]) {
                        sh "docker push ${IMAGE_NAME}"
                    }
                }
            }
        }

        // stage('Cleanup Old Containers and Images') {
        //     steps {
        //         script {
        //             sh """
        //                 sshpass -p '${TARGET_PASSWORD}' ssh -T -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_SERVER} '
                        
        //                 # Stop and remove old containers based on the image
        //                 docker ps -a --filter "ancestor=arpits2931/rcv_daas_frontend:latest" --format "{{.ID}}" | xargs -r docker stop
        //                 docker ps -a --filter "ancestor=arpits2931/rcv_daas_frontend:latest" --format "{{.ID}}" | xargs -r docker rm
                        
        //                 # Remove old images
        //                 docker images "arpits2931/rcv_daas_frontend:latest" --format "{{.ID}}" | xargs -r docker rmi -f
        //                 '
        //             """
        //         }
        //     }
        // }

        stage('Deploy Using Docker Compose') {
            steps {
                script {
                    sh """
                        sshpass -p '${TARGET_PASSWORD}' ssh -T -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_SERVER} '
                        
                        # Change to the deployment directory and start services
                        cd ${DEPLOY_PATH} && docker-compose up -d
                        '
                    """
                }
            }
        }

        stage('Final Stage') {
            steps {
                echo 'All stages have been completed successfully.'
            }
        }
    }
}
