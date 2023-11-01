node {
    def ansible_ip   = '172.31.19.78'
    def k8s_server_ip = '172.31.53.170'
    
    stage('Git checkout') {
        git 'https://github.com/Jaro233/k8s-ci-cd.git'
    }
    stage('Sending Dockerfile to Ansible server over ssh'){
        sshagent(['ansible-login']) {
            sh "ssh -o StrictHostKeyChecking=no ubuntu@${ansible_ip}"
            sh "scp /var/lib/jenkins/workspace/k8s-pipeline/* ubuntu@${ansible_ip}:/home/ubuntu"
        }
    }
    stage("Docker build image"){
         sshagent(["ansible-login"]) {
            sh "ssh -o StrictHostKeyChecking=no ubuntu@${ansible_ip} cd /home/ubuntu/"
            sh "ssh -o StrictHostKeyChecking=no ubuntu@${ansible_ip} docker build -t ${JOB_NAME}:v1.${BUILD_ID} ."
        }
    }
    stage("Docker image tagging"){
        sshagent(["ansible-login"]) {
            sh "ssh -o StrictHostKeyChecking=no ubuntu@${ansible_ip} cd /home/ubuntu/"
            sh "ssh -o StrictHostKeyChecking=no ubuntu@${ansible_ip} docker image tag ${JOB_NAME}:v1.${BUILD_ID} j4ro123/${JOB_NAME}:v1.${BUILD_ID}"
            sh "ssh -o StrictHostKeyChecking=no ubuntu@${ansible_ip} docker image tag ${JOB_NAME}:v1.${BUILD_ID} j4ro123/${JOB_NAME}:latest"
        }
    }
    stage("Push docker images to dockerhub"){
        sshagent(["ansible-login"]) {
            withCredentials([string(credentialsId: "docker_passw", variable: "docker_passw")]) {
                sh "ssh -o StrictHostKeyChecking=no ubuntu@${ansible_ip} docker login -u j4ro123 -p ${docker_passw}"
                sh "ssh -o StrictHostKeyChecking=no ubuntu@${ansible_ip} docker image push j4ro123/${JOB_NAME}:v1.${BUILD_ID}"
                sh "ssh -o StrictHostKeyChecking=no ubuntu@${ansible_ip} docker image push j4ro123/${JOB_NAME}:latest"
                sh "ssh -o StrictHostKeyChecking=no ubuntu@${ansible_ip} docker rmi j4ro123/${JOB_NAME}:latest j4ro123/${JOB_NAME}:v1.${BUILD_ID}"
            }
        }
    }
    stage("Copy files from jenkins to K8s server"){
            sshagent(["k8s_node"]) {
                sh "ssh -o StrictHostKeyChecking=no ubuntu@${k8s_server_ip}"
                sh "scp /var/lib/jenkins/workspace/k8s-pipeline/* ubuntu@${k8s_server_ip}:/home/ubuntu"
            }
    }
    stage("K8s deployment with ansible"){
        sshagent(["ansible-login"]) {
                sh "ssh -o StrictHostKeyChecking=no ubuntu@${ansible_ip} cd /home/ubuntu/"
                sh "ssh -o StrictHostKeyChecking=no ubuntu@${ansible_ip} ansible-playbook ansible.yml"
        }
    }
}
