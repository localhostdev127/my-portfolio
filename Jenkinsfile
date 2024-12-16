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
    }
    
    // parameters {
    //     string(name: 'APP_DOCKER_TAG_NAME', defaultValue: '', description: 'Setting docker image for latest push')
    // }
    

    stages {

        stage("Workspace cleanup"){
            steps{
                script{
                    cleanWs()
                }
            }
        }
        
        stage('Clone Repository') {
            steps {
                git branch: 'master', url:GITHUB_URL , credentialsId: 'github-credentials' 
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
                sh "docker build --build-arg API_KEY=${params.APP_DOCKER_TAG_NAME} -t ${DOCKER_USER_NAME}/${PROJECT_NAME}:${params.APP_DOCKER_TAG_NAME} ."
            }
        }

        stage("Docker: Push to DockerHub"){
            steps{
                
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    sh '''
                        echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin
                    '''
                }
                sh "docker push ${DOCKER_USER_NAME}/${PROJECT_NAME}:${params.APP_DOCKER_TAG_NAME}"
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
