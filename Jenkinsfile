
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


pipeline { 
    agent any

    environment {
        TARGET_SERVER = '172.16.0.103'
        TARGET_USER = 'root'
        TARGET_PASSWORD = 'Teamw0rk@1'
        DEPLOY_PATH = 'Horizan_reports'
    }

    stages {
        stage('Git Checkout') {
            steps {
               script {
    sh """
        sshpass -p '${TARGET_PASSWORD}' ssh -T -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_SERVER} '
        
        # Define variables
        REPO_URL="https://github.com/thinkcloud-in/guacamole_client.git"
        CREDENTIALS_ID="rcv-git"
        IMAGE_NAME="arpits2931/gucamole_openid:1.1.1"
        WORK_DIR="temp_guacamole_build"

        # Create a directory
        mkdir -p $WORK_DIR

        # Change to the directory
        cd $WORK_DIR

        # Clone the repository
        git clone --single-branch --branch main $REPO_URL .

        # Build the Docker image
        docker build -t $IMAGE_NAME .

        # Cleanup: Remove the directory after the build
        cd ..
        rm -rf $WORK_DIR
        '
    """
}
            }
        }

        stage('Docker Build & Tag') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'dokcer-id']) { // Use your Docker Hub credentials ID
                        sh 'docker build -t   arpits2931/gucamole_openid:1.1.1  .'
                    }
                }
            }
        }

       stage('Docker Push') {
            steps {
                script {
                    withDockerRegistry([credentialsId: 'dokcer-id']) { // Use the same credentials ID
                        sh 'docker push arpits2931/gucamole_openid:1.1.1'
                    }
                }
            }
        }

        stage('Run pwd Command') {
            steps {
              script {
                    sh """
                        sshpass -p '${TARGET_PASSWORD}' ssh -T -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_SERVER} '
                        
                        docker ps -a --filter "ancestor=arpits2931/rcv_daas_frontend:latest" --format "{{.ID}}" | xargs -r docker stop;
                        docker ps -a --filter "ancestor=arpits2931/rcv_daas_frontend:latest" --format "{{.ID}}" | xargs -r docker rm;
                        docker images "arpits2931/rcv_daas_frontend:latest" --format "{{.ID}}" | xargs -r docker rmi -f;
                        '
                    """
                }
            }
        }

        stage('Change Directory and Run Docker') {
            steps {
                script {
                    sh "sshpass -p '${TARGET_PASSWORD}' ssh -T -o StrictHostKeyChecking=no ${TARGET_USER}@${TARGET_SERVER} 'cd ${DEPLOY_PATH} && docker-compose up -d'"
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
