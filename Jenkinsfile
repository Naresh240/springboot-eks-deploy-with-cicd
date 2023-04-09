pipeline {
    agent any
    stages {
        stage("SCM"){
            steps{
                git branch: 'master',
                    credentialsId: 'github-creds', 
                    url: 'https://github.com/Naresh240/springboot-eks-deploy-with-cicd.git'
            }
        }
        stage("Build"){
            steps{
                sh "mvn clean install"
            }
        }
        stage("Docker Image"){
            steps{
                sh "docker build -t springboothelloeks:${BUILD_NUMBER} ."
            }
        }
        stage("Pushing Image"){
            steps{
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'password', usernameVariable: 'username')]) {
                    sh '''
                        docker login -u ${username} -p ${password}
                        docker tag springboothelloeks:${BUILD_NUMBER} ${username}/springboothelloeks:${BUILD_NUMBER}
                        docker push ${username}/springboothelloeks:${BUILD_NUMBER}
                    '''
                }
            }
        }
        stage("Connect to k8s cluster"){
            steps{
                sh "aws eks update-kubeconfig --region us-east-1 --name eksdemo"
            }
        }
        stage("Deploy Manifest files"){
            steps{
                dir('k8s') {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', passwordVariable: 'password', usernameVariable: 'username')]) {
                        sh '''
                            cat deployment.yaml | sed "s/{{reponame}}/${username}/g" | sed "s/{{ImageTag}}/${BUILD_NUMBER}/g" | kubectl apply -f -
                            kubectl apply -f service.yaml
                        '''
                    }
                }
            }
        }
    }
}