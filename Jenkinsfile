pipeline { 
    agent any 
 
    environment { 
        DOCKERHUB_REPO = 'najmura/demo-app' 
        MANIFEST_REPO  = 'https://github.com/nazamkhan78626/k8s-manifests.git' 
        MANIFEST_BRANCH = 'main' 
        APP_NAME = 'demo-app' 
    } 
 
    stages { 
        stage('1. Checkout Source Code') { 
            steps { 
                checkout scm 
            } 
        } 
 
        stage('2. Maven Clean Test Package') { 
            steps { 
                sh 'mvn clean test package' 
            } 
        } 
 
        stage('3. SonarQube Code Quality Scan') { 
            steps { 
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) { 
                    sh ''' 
                        mvn sonar:sonar \\ 
                        -Dsonar.projectKey=demo-app \\ 
                        -Dsonar.projectName=demo-app \\ 
                        -Dsonar.host.url=http://3.110.223.42:9000 \\ 
                        -Dsonar.login=$SONAR_TOKEN 
                    ''' 
                } 
            } 
        } 
 
        stage('4. Docker Build Image') { 
            steps { 
                sh 'docker build -t $DOCKERHUB_REPO:$BUILD_NUMBER .' 
                sh 'docker tag $DOCKERHUB_REPO:$BUILD_NUMBER $DOCKERHUB_REPO:latest' 
            } 
        } 
 
        stage('5. DockerHub Login and Push') { 
            steps { 
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) { 
                    sh ''' 
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin 
                        docker push $DOCKERHUB_REPO:$BUILD_NUMBER 
                        docker push $DOCKERHUB_REPO:latest 
                    ''' 
                } 
            } 
        } 
 
        stage('6. Update Kubernetes Manifest Repo') { 
            steps { 
                withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) { 
                    sh ''' 
                        rm -rf k8s-manifests 
                        git clone https://$GIT_USER:$GIT_TOKEN@github.com/nazamkhan78626/k8s-manifests.git 
                        cd k8s-manifests 
                        sed -i "s|image: .*|image: $DOCKERHUB_REPO:$BUILD_NUMBER|g" deployment.yaml 
                        git config user.email "jenkins@example.com" 
                        git config user.name "jenkins-ci" 
                        git add deployment.yaml 
                        git commit -m "Update image to $DOCKERHUB_REPO:$BUILD_NUMBER" || echo "No manifest change found" 
                        git push origin $MANIFEST_BRANCH 
                    ''' 
                } 
            } 
        } 
    } 
 
    post { 
        success { 
            echo 'Pipeline completed successfully. ArgoCD will deploy the updated image.' 
        } 
        failure { 
            echo 'Pipeline failed. Check Jenkins console output.' 
        } 
    } 
} 
