pipeline{
        agent any
        stages{
            stage('Build images on Jenkins'){
                steps{
                    sh '''
                    docker build -t mtkg/trio-nginx-img:latest ./nginx
                    docker build -t mtkg/trio-nginx-img:${BUILD_NUMBER} ./nginx
                    docker build -t mtkg/trio-flask-app-img:latest ./flask-app
                    docker build -t mtkg/trio-flask-app-img:${BUILD_NUMBER} ./flask-app
                    docker build -t mtkg/trio-db-img:latest ./db
                    docker build -t mtkg/trio-db-img:${BUILD_NUMBER} ./db
                    '''
                }
            }
            stage('Push images to Docker Hub'){
                steps{
                    sh '''
                    docker push mtkg/trio-nginx-img
                    docker push mtkg/trio-nginx-img:${BUILD_NUMBER}
                    docker push mtkg/trio-flask-app-img
                    docker push mtkg/trio-flask-app-img:${BUILD_NUMBER}
                    docker push mtkg/trio-db-img
                    docker push mtkg/trio-db-img:${BUILD_NUMBER}
                    '''
                }
            }
            stage('Clean up old images on Jenkins'){
                steps{
                    sh '''
                    #Delete build number and latest images:
                    docker rmi mtkg/trio-nginx-img
                    docker rmi mtkg/trio-nginx-img:${BUILD_NUMBER}
                    docker rmi mtkg/trio-flask-app-img
                    docker rmi mtkg/trio-flask-app-img:${BUILD_NUMBER}
                    docker rmi mtkg/trio-db-img
                    docker rmi mtkg/trio-db-img:${BUILD_NUMBER}
                    '''
                }
            }
            stage('Deploy containers to app server'){
                steps{
                    sh '''
                    #SSH onto app server as jenkins user:
                    ssh -i "~/.ssh/id_rsa_new_vm" jenkins@10.154.0.27 << EOF

                    #Pull latest images from DockerHub:
                    docker image pull mtkg/trio-nginx-img
                    docker image pull mtkg/trio-flask-app-img
                    docker image pull mtkg/trio-db-img

                    #Create network if it doesn't already exist:
                    docker network inspect trio && sleep 1 || docker network create trio

                    #Stop and remove existing containers:
                    docker stop mysql && (docker rm mysql) || (docker rm mysql && sleep 1 || sleep 1)
                    docker stop flask-app && (docker rm flask-app) || (docker rm flask-app && sleep 1 || sleep 1)
                    docker stop mynginx && (docker rm mynginx) || (docker rm mynginx && sleep 1 || sleep 1)
                    #docker rm -f mysql || true
                    #docker rm -f flask-app || true
                    #docker rm -f mynginx || true

                    #Run containers from images:
                    docker run -d --network trio --name mysql mtkg/trio-db-img:latest
                    docker run -d --network trio --name flask-app mtkg/trio-flask-app-img:latest
                    docker run -d -p 80:80 --network trio --name mynginx mtkg/trio-nginx-img:latest

                    #RANDOM COMMENT TO TRIGGER BUILD - 2

                    '''
                }
            }
        }
}