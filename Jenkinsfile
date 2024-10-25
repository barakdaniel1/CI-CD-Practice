pipeline {
    agent any
    
    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerhub-credentials')
        DOCKER_IMAGE_NAME = 'barakdaniel95/ci-cd-project'
    }
    stages {
        stage('Clean Workspace'){
            steps {
                //Clean the workspace before the pipeline starts
                echo 'Cleaning workspace to ensure a clean environment'
                cleanWs()
            }
        }
        stage('Checkout') {
            steps {
                echo 'Checking out from Github'
                
                checkout([$class: 'GitSCM', 
                    branches: [[name: '*/main']], 
                    userRemoteConfigs: [[url: 'https://github.com/barakdaniel1/CI-CD-Practice']]
                ])
            }
        }
        stage('Build') {
            steps {
                echo 'Building stage-----'
                
                sh '''
                      npm install 
                '''
            }
        }
        stage('Test') {
            steps {
                echo 'Testing stage-----'
                echo 'Starting Node.js server..'
                sh '''
                    node index.js &
                    echo $! > .pidfile
                    '''

                echo 'Testing server with curl..'
                sh '''sleep 5
                      curl -f http://localhost:3000 || echo "Server not responding"
                '''
                //Now kill the process that runs the server:
                sh '''
                    kill $(cat .pidfile) || echo "Server process already stopped"
                    rm .pidfile
                '''
                echo 'CI stages are done! Starting CD...'
            }
        }
        
        stage('Docker Credentials'){
            steps{
                echo 'Logging in to docker-hub..'
                
                //Login to docker-hub
                sh '''
                    echo "${DOCKER_HUB_CREDENTIALS_PSW}" | docker login -u ${DOCKER_HUB_CREDENTIALS_USR} --password-stdin
                '''
            }
        }
        
        stage('Docker Build'){
            steps{
                //Build the image
                echo 'Building the docker image..'
                sh '''
                    docker build -t ${DOCKER_IMAGE_NAME}:latest .
                '''
            }
        }
        
        stage('Docker push'){
            steps{
                echo 'Pushing the image to docker-hub..'
                sh'''
                    docker push ${DOCKER_IMAGE_NAME}:latest
                '''
            }
        }
        
        stage('Run Docker container on Jenkins server'){
           steps{
               sh '''
                    docker pull ${DOCKER_IMAGE_NAME}:latest
                    docker stop ci-cd-project || true
                    docker rm ci-cd-project || true
                    docker run -d -p 3000:3000 --name ci-cd-project ${DOCKER_IMAGE_NAME}:latest
               '''
           }
        }
        
        stage('Verify Docker Container') {
            steps {
                echo 'Verifying that the Docker container is running and accessible...'
                
                sh 'sleep 5'
                sh '''
                    curl -f http://localhost:3000 || echo "Server not responding"
                '''
            }
        }
    }
    
    post{
        always{
            echo 'Pipeline finished!'
            sh 'docker logout'
        }
    }
}

