pipeline {
    agent any
    stages {
        stage('Prepare param') {
            steps {
                script{
                    cleanWs()
                    checkout scm
                    project_id = "helpful-girder-423003-n6"
                    cluster = "example-cluster"
                    zone = "us-central1-a"
                    branch = "${JOB_BASE_NAME}"                    
                    if ("${branch}" == "master"){
                        port = '3000'
                    } else{
                        port = '4000'
                    }
                    version = "${BUILD_NUMBER}"
                }
            }
        }
        stage('BuildImg') {
            steps {
                script{
                    withCredentials([usernamePassword(credentialsId: "jenkins-slave", usernameVariable: "USER", passwordVariable: "PASS")]){
                        withCredentials([usernamePassword(credentialsId: "dockerhub", usernameVariable: "USERNAME", passwordVariable: "PASSWORD")]){
                            sh"""
                            sshpass -p '${PASS}' ssh -o StrictHostKeyChecking=no -l ${USER} 34.87.59.112 'rm -rf /tmp/test && mkdir -p /tmp/test && cd /tmp/test && git init && git clone https://github.com/voanchalee/node-js-postgresql-crud-example.git'
                            sshpass -p '${PASS}' ssh -o StrictHostKeyChecking=no -l ${USER} 34.87.59.112 'cd /tmp/test && cd node-js-postgresql-crud-example && git checkout ${branch} && git status'
                            sshpass -p '${PASS}' ssh -o StrictHostKeyChecking=no -l ${USER} 34.87.59.112 'cd /tmp/test && cd node-js-postgresql-crud-example && docker login -u ${USERNAME} -p ${PASSWORD}; docker build -t anchaleev/nodendb:${version} .;docker push anchaleev/nodendb:${version}'
                            """
                        }
                    }
                }
            }
        }
        stage('Deploy Service') {
            steps {
                script{
                    sh"""
                        gcloud container clusters get-credentials ${cluster} --zone ${zone} --project ${project_id}
                        kubectl get ns
                        sed -i 's/image_version/${version}/g' nodejs.yml
                        cat nodejs.yml
                        kubectl apply -f nodejs.yml
                        kubectl apply -f hpa.yml
                    """
                }
            }
        }
        stage('Attach PublicIP') {
            steps {
                script{
                    listpods = sh(script: "kubectl get --no-headers=true pods -o name | awk -F \"/\" '{print \$2}' |grep nodejs",returnStdout:true).trim()
                    echo "${listpods}"
                    pods = "${listpods}".split("\\r?\\n")
                    pods.each { pod ->
                        sh"kubectl expose pod ${pod} --type=LoadBalancer --port=${port} --target-port=8080"
                    }
                    sh"kubectl get svc -w --request-timeout=40s"
                }
            }
        }
    }
}
