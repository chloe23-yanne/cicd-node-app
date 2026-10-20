pipeline {

    agent any


    environment {

        DOCKER_IMAGE_NAME = "chloe23yanne/test-node"

        DOCKER_IMAGE_TAG = "latest"

        ANSIBLE_SERVER = "cloee@10.74.81.1"

    }


    stages {


        stage('Checkout') {

            steps {

                git branch: 'main',
                    url: 'https://github.com/chloe23-yanne/cicd-node-app.git'

            }

        }


        stage('Remove Existing Local Image') {

            steps {

                sh '''

                    docker rmi \
                    ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} \
                    || true

                '''

            }

        }


        stage('Build Docker Image') {

            steps {

                sh '''

                    docker build \
                    --rm \
                    -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} .

                '''

            }

        }


        stage('Push Docker Image to Docker Hub') {

            steps {

                withCredentials([

                    usernamePassword(

                        credentialsId: 'docker-hub-credentials',

                        usernameVariable: 'DOCKERHUB_USERNAME',

                        passwordVariable: 'DOCKERHUB_PASSWORD'

                    )

                ]) {

                    sh '''

                        echo "${DOCKERHUB_PASSWORD}" \
                        | docker login \
                        -u "${DOCKERHUB_USERNAME}" \
                        --password-stdin


                        docker push \
                        ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}

                    '''

                }

            }

        }


        stage('SonarQube Analysis') {

            steps {

                withSonarQubeEnv('sonar_qube') {

                    sh '''

                        sonar-scanner \
                        -Dsonar.projectKey=test-node \
                        -Dsonar.sources=. \
                        -Dsonar.exclusions=node_modules/**

                    '''

                }

            }

        }


        stage('Run Ansible Playbook') {

            steps {

                sh '''

                    ssh \
                    -o StrictHostKeyChecking=no \
                    ${ANSIBLE_SERVER} \
                    "cd ~/ansible-lab && ~/ansible-venv/bin/ansible-playbook deploy-node.yml"

                '''

            }

        }


        stage('Deploy') {

            steps {

                echo 'Application deployed successfully'

            }

        }

    }


    post {


        success {

            echo 'PIPELINE SUCCESS'

            echo 'Application successfully deployed'

        }


        failure {

            echo 'PIPELINE FAILED'

            echo 'Check Jenkins Console Output'

        }

    }

}
