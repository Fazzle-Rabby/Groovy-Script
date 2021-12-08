# Groovy-Script  (Scripted Pipeline)

Developer > Github > Jenkins > Docker-Host

# Build Triggers

GitHub hook trigger for GITScm polling

# Pipeline

// Definition
pipeline script

// script
node{
    stage("Pull Sourcecode from Github"){
        git 'https://github.com/Fazzle-Rabby/Docker-Project.git'
    }
    
    stage("Build Dockerfile"){
    sh  'docker image build -t $JOB_NAME:v1.$BUILD_ID .'
    sh 'docker image tag $JOB_NAME:v1.$BUILD_ID  fazzlerabby/$JOB_NAME:v1.$BUILD_ID'
    sh 'docker image tag $JOB_NAME:v1.$BUILD_ID  fazzlerabby/$JOB_NAME:latest'
    }
    
    stage("Push image Dockerhub"){
        withCredentials([string(credentialsId: 'pass', variable: 'pass')]) {
    // some block
    sh 'docker login -u fazzlerabby -p ${pass}'
    sh 'docker image push fazzlerabby/$JOB_NAME:v1.$BUILD_ID'
    sh 'docker image push fazzlerabby/$JOB_NAME:latest'
    sh 'docker image rmi $JOB_NAME:v1.$BUILD_ID fazzlerabby/$JOB_NAME:v1.$BUILD_ID fazzlerabby/$JOB_NAME:latest'
}
    }
    
    stage("Deployment of docker container"){
        def dockerrun = 'docker run -p 8080:80 -d --name parvez fazzlerabby/scripted-pipeline-demo:latest'
        def dockerrm = 'docker container rm -f parvez'
        def dockerimage = 'docker image rmi fazzlerabby/scripted-pipeline-demo'
      sshagent(['docker']) {
    // some block
    sh "ssh -o StrictHostKeyChecking=no  ec2-user@172.31.30.32 ${dockerrm}"
    sh "ssh -o StrictHostKeyChecking=no  ec2-user@172.31.30.32 ${dockerimage}"
    sh "ssh -o StrictHostKeyChecking=no  ec2-user@172.31.30.32 ${dockerrun}"
}  
    }
}
