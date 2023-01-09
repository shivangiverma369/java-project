# java-project
Deployment on k8s using ci/cd pipeline

what we are doing:-
we are going to deploy deploy app/service on k8s using jenkins pipeline, for this i have a repository
in my git named as java-project 
prerequisite:-
jenkins server installed plugin ssh-agent, maven integration and configur with git, java, maven etc
and in mange credentials configure th credentials of github, dockerhub.

So all set here we need to do is go to jenkins server create a pipeline – set github weebhook for sync 
gave github project url, in pipeline script start to write th script as follow:-

pipeline{
    agent any
    stages {
        stage('Build Maven') {
            steps{
               checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'java-project', url: 'https://github.com/shivangiverma369/java-project.git']]])              //go to pipeline script choose checkout: check out from version control and 
                                           add the credentials of your github.
             
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                  sh 'docker build -t vermashivangi/nodeapp:latest .'
                }
            }
        }
        stage('Deploy Docker Image') {
            steps {
                script {
                 withCredentials([string(credentialsId: 'docker2', variable: 'docker')]) {    
         //in pipeline script go to the with credentials- bind the credentials to the variable  and choose the secret text add the dockerhub credential accordingly


                    sh 'docker login -u vermashivangi -p ${docker}'
                 }  
                 sh 'docker push vermashivangi/nodeapp:latest'
                }
            }
        }
    
    stage('Deploy App on k8s') {
      steps {                                                                                     
         sshagent(['jenkins']) {      //in pipeline script go to the ssh-agent give the username of k8s cluster where we need to ssh and private key of te host where your jenkins server is running (don’t forget to copy public key from your host(like from which your jenkins server is running) to k8s cluster host otherwise jenkins give error of permission denied) .                                                                    
           sh "scp -o StrictHostKeyChecking=no nodejsapp.yaml main2@192.168.122.5:/home/main2"
           script {
                try{
                    sh "ssh main2@192.168.122.5 kubectl apply -f ."
                }catch(error){
                    sh "ssh main2@192.168.122.5 kubectl create -f ."
            }
}
        }
      
    }
    }
    }
}








         


