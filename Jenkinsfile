node{
    stage("git checkout"){
        git branch: 'main', url: 'https://github.com/Mohd-Sayeedul-Hoda/blogging.git'
    }
    stage('sending dockerfile to Ansible server over ssh'){
        sshagent(['ansible_demo']) {
             sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.46.84'
             sh 'scp -r /var/lib/jenkins/workspace/pipeline-demo/* ubuntu@172.31.46.84:/home/ubuntu'
        }
    }
    stage('Docker image building'){
        sshagent(['ansible_demo']) {
             sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.46.84 cd /home/ubuntu/'
             sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.46.84 docker image build -t $JOB_NAME:v1.$BUILD_ID .'
            
        }
    }
    stage('Docker image tagging'){
        sshagent(['ansible_demo']) {
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.46.84 cd /home/ubuntu/'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.46.84 docker image tag $JOB_NAME:v1.$BUILD_ID sayeedulhoda/$JOB_NAME:v1.$BUILD_ID' 
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.46.84 docker image tag $JOB_NAME:v1.$BUILD_ID sayeedulhoda/$JOB_NAME:latest'        
            }
    }
    stage('push Docker image to Dockerhub'){
        sshagent(['ansible_demo']) {
            withCredentials([usernamePassword(credentialsId: 'DockerhubPassword', passwordVariable: 'dockerhubpass', usernameVariable: 'sayeedulhoda')]) {
                sh "ssh -o StrictHostKeyChecking=no ubuntu@172.31.46.84 docker login -u sayeedulhoda -p ${dockerhubpass}"
                sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.46.84 docker push sayeedulhoda/$JOB_NAME:v1.$BUILD_ID'
                sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.46.84 docker push sayeedulhoda/$JOB_NAME:latest'
                sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.46.84 docker rmi sayeedulhoda/$JOB_NAME:v1.$BUILD_ID sayeedulhoda/$JOB_NAME:latest'
            }
        }
    }
    stage('Copy file from k8 file from jenkins to k8 server'){
            sshagent(['ansible_demo']) {
                sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.40.184 cd /home/ubuntu'
                sh 'scp /var/lib/jenkins/workspace/pipeline-demo/Deployment.yml ubuntu@172.31.40.184:/home/ubuntu'
                sh 'scp /var/lib/jenkins/workspace/pipeline-demo/Service.yml ubuntu@172.31.40.184:/home/ubuntu'

            }
        }
    stage('running ansible playbook in ansible server'){
        sshagent(['ansible_demo']) {
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.46.84 cd /home/ubuntu'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.46.84 ansible -m ping nodes'
            sh 'ssh -o StrictHostKeyChecking=no ubuntu@172.31.46.84 ansible-playbook ansible.yml'
        }
    }
    }