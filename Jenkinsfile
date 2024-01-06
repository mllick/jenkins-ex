pipeline {
    agent any

    environment {
        DOCKER_ID = "sarrlick"
        DOCKER_IMAGE = "cast-movie"  // Mettez à jour avec votre nom d'utilisateur Docker Hub
        DOCKER_TAG = "v.${BUILD_ID}.0"
        KUBECONFIG = credentials("config")
        DOCKER_HUB_PASS = credentials("DOCKER_HUB_PASS")  // Assurez-vous d'avoir défini les identifiants Docker Hub dans Jenkins
    }

    stage('Install Docker Compose') {
    
    stages {
        stage('Install Docker Compose') {
            steps {
                script {
                    sh '''
                        sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
                        sudo chmod +x /usr/local/bin/docker-compose
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    sh '''
                        docker-compose -f docker-compose.yml build cast_movie
                    '''
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    sh '''
                        docker login -u $DOCKER_ID -p $DOCKER_HUB_PASS
                        docker-compose -f docker-compose.yml push cast_movie
                    '''
                }
            }
        }

        stage('Deployment et Tests') {
            steps {
                script {
                    sh '''
                        docker-compose -f docker-compose.yml up -d
                        sleep 10  # Peut être ajusté en fonction de votre besoin
                        curl localhost:8080
                    '''
                }
            }
        }

        stage('Deploy to Dev Environment') {
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp mon-chart-helm/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install cast-movie-dev ./cast-movie --values=values.yml --namespace dev
                    '''
                }
            }
        }

        stage('Deploy to QA Environment') {
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp mon-chart-helm/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install cast-movie-qa ./cast-movie --values=values.yml --namespace qa
                    '''
                }
            }
        }

        stage('Deploy to Staging Environment') {
            steps {
                script {
                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp mon-chart-helm/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install cast-movie-staging ./cast-movie --values=values.yml --namespace staging
                    '''
                }
            }
        }

        stage('Deploy to Prod Environment') {
            steps {
                script {
                    timeout(time: 15, unit: "MINUTES") {
                        input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    }

                    sh '''
                        rm -Rf .kube
                        mkdir .kube
                        cat $KUBECONFIG > .kube/config
                        cp mon-chart-helm/values.yaml values.yml
                        sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                        helm upgrade --install cast-movie-prod ./cast-movie --values=values.yml --namespace prod
                    '''
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    sh '''
                        docker-compose -f docker-compose.yml down
                    '''
                }
            }
        }
    }
}
