pipeline {
    agent any

    options {
        disableConcurrentBuilds() 
        timestamps() 
    }

    environment{
        PROJECT_NAME = "ji-portfolio"
        DOCKER_USER_NAME = "jamirul"
        DOCKER_TAG_NAME="latest"
        GITHUB_URL='https://github.com/localhostdev127/my-portfolio.git'
        SSH_CREDENTIALS_ID = 'remote-ssh-credentials'
        DOCKER_CREDENTIALS_ID = 'docker-hub-credentials'
    }
    
    parameters {
        string(name: 'REMOTE_HOST_ADDRESS', defaultValue: '', description: 'Remote host ip address')
    }
    

    stages {
        stage("Validate Parameters") {
            steps {
                script {
                    if (params.REMOTE_HOST_ADDRESS == '') {
                        error("REMOTE_HOST_ADDRESS must be provided.")
                    }
                }
            }
        }
        stage("Workspace cleanup"){
            steps{
                script{
                    cleanWs()
                }
            }
        }
        
        stage('Clone Repository') {
            steps {
                git branch: 'main', url:GITHUB_URL , credentialsId: 'github-credentials' 
            }
        }

        stage('Docker: Clear all cached Images'){
            steps{
                script {
                    
                    sh '''
                        docker rmi -f $(docker images -aq) || echo "No images to remove"
                    '''
                }
            }
        }

        stage('Docker: Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_USER_NAME}/${PROJECT_NAME}:${DOCKER_TAG_NAME} ."
            }
        }

        stage("Docker: Push to DockerHub"){
            steps{
                
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh '''
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                    '''
                }
                sh "docker push ${DOCKER_USER_NAME}/${PROJECT_NAME}:${DOCKER_TAG_NAME}"
            }
        }


        stage('Test') {
            steps {
                echo 'skipping test ...'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                // Add your deployment steps here
                    script {
                    def remote = [:]
                    withCredentials([
                        sshUserPrivateKey(credentialsId: SSH_CREDENTIALS_ID, 
                                          keyFileVariable: 'identity', 
                                          passphraseVariable: '', 
                                          usernameVariable: 'userName'),
                        usernamePassword(credentialsId: DOCKER_CREDENTIALS_ID, 
                                         usernameVariable: 'DOCKER_USER', 
                                         passwordVariable: 'DOCKER_PASS')
                    ]) {
                        remote.name = "node-1"
                        remote.host = params.REMOTE_HOST_ADDRESS
                        remote.allowAnyHosts = true
                        remote.user = userName
                        remote.identityFile = identity
                        
                        // SSH into the server and execute Docker commands
                        sshCommand remote: remote, command: '''
                            mkdir hello
                            # Stop all running containers
                            docker ps -q | xargs -r docker stop

                            # Remove all containers
                            docker ps -aq | xargs -r docker rm

                            # Remove all images
                            docker images -q | xargs -r docker rmi -f

                            # Login to Docker Hub
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                            # Pull the latest Docker image
                            docker pull "$DOCKER_USER_NAME"/"$PROJECT_NAME":"$DOCKER_TAG_NAME"

                            # Run the Docker container
                            docker run -p 80:80 -d "$DOCKER_USER_NAME"/"$PROJECT_NAME":"$DOCKER_TAG_NAME"
                        '''
                    }
                }

            }
        }
    }

    post {
        always {
            echo 'Pipeline finished!'
        }
        success {
            echo 'Build succeeded!'
        }
        failure {
            echo 'Build failed.'
        }
    }
}
