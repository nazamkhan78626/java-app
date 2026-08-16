pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = 'najmura/demo-app'
        MANIFEST_REPO = 'https://github.com/nazamkhan78626/k8s-manifests.git'
        MANIFEST_BRANCH = 'main'
        APP_NAME = 'demo-app'
    }

    stages {

        stage('1. Checkout Source Code') {
            steps {
                checkout scm
            }
        }

        stage('2. Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

       stage('3. SonarQube Analysis') {
    steps {
        withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
            sh '''
                mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.11.0.3922:sonar \
                -Dsonar.projectKey=demo-app \
                -Dsonar.projectName=demo-app \
                -Dsonar.host.url=http://13.201.19.146:9000 \
                -Dsonar.token=${SONAR_TOKEN}
            '''
        }
    }
}
        stage('4. Docker Build') {
            steps {
                sh 'docker build -t $DOCKERHUB_REPO:$BUILD_NUMBER .'
                sh 'docker tag $DOCKERHUB_REPO:$BUILD_NUMBER $DOCKERHUB_REPO:latest'
            }
        }

        stage('5. DockerHub Push') {
            steps {
                withCredentials([usernamePassword(
    credentialsId: 'dockerhub-creds',
    usernameVariable: 'DOCKER_USER',
    passwordVariable: 'DOCKER_PASS'
)]) {
    sh '''
        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
        docker push $DOCKERHUB_REPO:$BUILD_NUMBER
        docker push $DOCKERHUB_REPO:latest
    '''
}

        stage('6. Update Manifest Repo') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-creds',
                                                  usernameVariable: 'GIT_USER',
                                                  passwordVariable: 'GIT_TOKEN')]) {

                    sh '''
                        rm -rf k8s-manifests

                        git clone https://$GIT_USER:$GIT_TOKEN@github.com/nazamkhan78626/k8s-manifests.git

                        cd k8s-manifests

                        sed -i "s|image: .*|image: najmura/demo-app:$BUILD_NUMBER|g" deployment.yaml

                        git config user.email "jenkins@example.com"
                        git config user.name "jenkins-ci"

                        git add deployment.yaml

                        git commit -m "Update image to build $BUILD_NUMBER" || true

                        git push origin main
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }
    }
}
