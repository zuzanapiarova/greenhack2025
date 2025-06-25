pipeline {
    agent any

    environment {
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        DOCKERHUB_USER='zuzanapiarova'
        BACKEND_IMAGE='backend-img'
        FRONTEND_IMAGE='frontend-img'
        EC2_PUBLIC_IP=credentials('EC2_PUBLIC_IP')
        FRONTEND_CONTAINER_NAME='frontend-container'
        BACKEND_CONTAINER_NAME='backend-container'
    }

    stages {

        stage('Clone repository') {
            steps {
                checkout scm
                //git branch:'main', url: 'https://github.com/zuzanapiarova/greenhack2025'
            }
        }

        // stage('Build backend image') {
        //     steps {
        //         dir('app/backend') {
        //             script {
        //                 sh "docker build -t $DOCKERHUB_USER/$BACKEND_IMAGE:$IMAGE_TAG ."
        //             }
        //         }
        //     }
        // }

        // stage('Test Backend') {
        //     steps {
        //         script {
        //             def backendTestContainer = "test-backend-${BUILD_NUMBER}"
        //             try {
        //                 sh "docker run -d --name $backendTestContainer -p 5000:5000 $DOCKERHUB_USER/$BACKEND_IMAGE:$IMAGE_TAG"
        //                 timeout(time: 30, unit: 'SECONDS') {
        //                     waitUntil {
        //                         def status = sh(
        //                             script: "curl -s -o /dev/null -w '%{http_code}' http://localhost:5000/health || true",
        //                             returnStdout: true
        //                         ).trim()
        //                         return (status == '200')
        //                     }
        //                 }
        //             } catch (e) {
        //                 error "backend test failed or container unhealthy: ${e.getMessage()}"
        //             } finally {
        //                 sh "docker stop ${backendTestContainer} || true"
        //                 sh "docker rm ${backendTestContainer} || true"
        //             }
        //         }
        //     }
        // }

        // stage ('Build frontend app') {
        //     steps {
        //          dir('app/frontend') {
        //             withCredentials([
        //                 string(credentialsId: 'REACT_APP_ZABAGED_220V', variable: 'REACT_APP_ZABAGED_220V'),
        //                 string(credentialsId: 'REACT_APP_ZABAGED_400V', variable: 'REACT_APP_ZABAGED_400V'), 
        //                 string(credentialsId: 'REACT_APP_OPENSTREETMAP_URL', variable: 'REACT_APP_OPENSTREETMAP_URL'),
        //                 string(credentialsId: 'REACT_APP_API_URL', variable: 'REACT_APP_API_URL'), 
        //                 ]) {
        //                 sh '''
        //                     # Write envs to .env file dynamically
        //                     echo "REACT_APP_API_URL=$REACT_APP_API_URL" > .env
        //                     echo "REACT_APP_OPENSTREETMAP_URL=$REACT_APP_OPENSTREETMAP_URL" >> .env
        //                     echo "REACT_APP_ZABAGED_220V=$REACT_APP_ZABAGED_220V" >> .env
        //                     echo "REACT_APP_ZABAGED_400V=$REACT_APP_ZABAGED_400V" >> .env

        //                     npm install
        //                     npm run build

        //                     rm .env
        //                     rm -rf node_modules
        //                 '''
        //             }
        //         }
        //     }
        // }

        // stage('Build frontend image') {
        //     steps {
        //         dir('app/frontend') {
        //             sh "docker build -t $DOCKERHUB_USER/$FRONTEND_IMAGE:$IMAGE_TAG ."
        //         }
        //     }
        // }

        // stage('Test Frontend') {
        //     steps {
        //         script {
        //             def frontendTestContainer = "test-frontend-${BUILD_NUMBER}"
        //             try {
        //                 sh "docker run -d --name ${frontendTestContainer} -p 80:80 $DOCKERHUB_USER/$FRONTEND_IMAGE:$IMAGE_TAG"
        //                 sleep(5)
                        
        //                 def output = sh(
        //                     script: "curl --fail http://localhost:80/health.json",
        //                     returnStdout: true
        //                 ).trim()
                        
        //                 if (!output.contains('"status": "ok"')) {
        //                     error "Frontend health check failed: ${output}"
        //                 }
        //             } catch (e) {
        //                 echo "Frontend test or health check failed: ${e.getMessage()}"
        //             } finally {
        //                 echo "Running cleanup in finally block for container ${frontendTestContainer}"
        //                 sh "docker stop $frontendTestContainer || true"
        //                 sh "docker rm $frontendTestContainer || true"
        //             }
        //         }
        //     }
        // }


        // stage('Push Docker images') {
        //     steps {
        //         withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
        //             script {
        //                 sh 'echo $PASS | docker login -u $USER --password-stdin'
        //                 // Tag and push as latest
        //                 sh "docker tag $DOCKERHUB_USER/$FRONTEND_IMAGE:$IMAGE_TAG $DOCKERHUB_USER/$FRONTEND_IMAGE:latest"
        //                 sh "docker push $DOCKERHUB_USER/$FRONTEND_IMAGE:latest"
        //                 sh "docker tag $DOCKERHUB_USER/$BACKEND_IMAGE:$IMAGE_TAG $DOCKERHUB_USER/$BACKEND_IMAGE:latest"
        //                 sh "docker push $DOCKERHUB_USER/$BACKEND_IMAGE:latest"
        //             }
        //         }
        //     }
        // }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ecs-ssh-key']) { // makes the private key available to any ssh or scp commands run within the block, works only inside sh steps
                    sh 'echo In SSH agent on EC2 instance'
                    // -o StrictHostKeyChecking=no flag tells SSH to skip host key verification, which avoids prompts like Are you sure you want to continue connecting (yes/no)?
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@$EC2_PUBLIC_IP << EOF 
echo Executing SSH block on EC2 instance

# Stop and remove old frontend container
docker stop $FRONTEND_CONTAINER_NAME || true
docker rm $FRONTEND_CONTAINER_NAME || true
# Stop and remove old backend container
docker stop $BACKEND_CONTAINER_NAME || true
docker rm $BACKEND_CONTAINER_NAME || true

# Optional cleanup
docker system prune -af

# Pull latest images
docker pull $DOCKERHUB_USER/$FRONTEND_IMAGE:latest
docker pull $DOCKERHUB_USER/$BACKEND_IMAGE:latest

# Run new frontend container
docker run -d --name $FRONTEND_CONTAINER_NAME --restart unless-stopped -p 80:80 $DOCKERHUB_USER/$FRONTEND_IMAGE:latest

# Run new backend container
docker run -d --name $BACKEND_CONTAINER_NAME --restart unless-stopped -p 5000:5000 $DOCKERHUB_USER/$BACKEND_IMAGE:latest
EOF
                    """
                }
            }
        }
    }

    post {
        always {
            sh 'docker system prune -af'
        }
    }
}