pipeline {
    agent any

    environment {
        IMAGE_NAME = "akshu20791/frontend"
        IMAGE_TAG = "v1.0.${BUILD_NUMBER}"
        APP_NAME = "frontend"

        // Replace with your actual ArgoCD server
       // ARGOCD_SERVER = "a18d4600cb2074f30a47cf7cc8440ccf-142765116.ap-southeast-2.elb.amazonaws.com:443"
          ARGOCD_SERVER = "ab79869818bbc49e887caad128fdcfdb-2008181742.us-east-1.elb.amazonaws.com:443"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                    docker build -t $IMAGE_NAME:$IMAGE_TAG .

                    echo $DOCKER_PASS | docker login \
                        -u $DOCKER_USER \
                        --password-stdin

                    docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Update GitOps Repo') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'github-bot-token',
                        variable: 'GIT_TOKEN'
                    )
                ]) {

                    sh '''
                    rm -rf k8s-config

                    git clone https://${GIT_TOKEN}@github.com/letslearn207/k8s-config

                    cd k8s-config

                    git config user.email "jenkins@bot.com"
                    git config user.name "jenkins-bot"

                    sed -i "s|image: .*|image: $IMAGE_NAME:$IMAGE_TAG|g" frontend/deployment.yaml

                    git add frontend/deployment.yaml

                    git commit -m "Update image to $IMAGE_TAG" || true

                    git push origin main
                    '''
                }
            }
        }

        stage('Trigger ArgoCD Sync') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJhcmdvY2QiLCJzdWIiOiJhZG1pbjphcGlLZXkiLCJuYmYiOjE3ODk4MDA5NjgsImlhdCI6MTc4OTgwMDk2OCwianRpIjoiMTEzZDFlYzgtMDgwZi00Y2U5LWE2YTctMGU0NWVhMzEzMzdhIn0.OmW1EeKUpXaqgw8taU8XML9mj0w8IymKE3NM5lbGlbA',
                        variable: 'ARGOCD_TOKEN'
                    )
                ]) {

                    sh '''
                    argocd app sync $APP_NAME \
                        --auth-token $ARGOCD_TOKEN \
                        --server $ARGOCD_SERVER \
                        --insecure \
                        --prune

                    argocd app wait $APP_NAME \
                        --auth-token $ARGOCD_TOKEN \
                        --server $ARGOCD_SERVER \
                        --insecure
                    '''
                }
            }
        }
    }

    post {

        success {
            echo 'Pipeline completed successfully'
        }

        failure {
            echo 'Pipeline failed'
        }

        always {
            cleanWs()
        }
    }
}
