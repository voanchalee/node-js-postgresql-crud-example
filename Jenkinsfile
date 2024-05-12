pipeline {
    agent any
    stages {
        stage('Prepare param') {
            steps {
                script{
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
                    withCredentials([usernamePassword(credentialsId: "dockerhub", usernameVariable: "USERNAME", passwordVariable: "PASSWORD")]){
                        sh"""
                        git checkout ${branch}
                        git status
                        #ls -la
                        #docker build -t anchaleev/nodendb:${version} .
                        #docker push anchaleev/nodendb:${version}
                        """
                    }
                }
            }
        }
        stage('Deploy Service') {
            steps {
                script{
                    sh"""
                        #/var/jenkins_home/google-cloud-sdk/bin/gcloud components install kubectl
                        /var/jenkins_home/google-cloud-sdk/bin/gcloud container clusters get-credentials ${cluster} --zone ${zone} --project ${project_id}
                        /var/jenkins_home/google-cloud-sdk/bin/kubectl get ns
                        /var/jenkins_home/google-cloud-sdk/bin/kubectl apply -f nodejs.yml
                        /var/jenkins_home/google-cloud-sdk/bin/kubectl apply -f hpa.yml
                    """
                }
            }
        }
        stage('Attach PublicIP') {
            steps {
                script{
                sh"""
                pod_name=\$(kubectl get pods -o=name | grep ${environment} | awk -F'/' '{print \$2}')
                /var/jenkins_home/google-cloud-sdk/bin/kubectl expose pod \$pod_name --type=LoadBalancer --port=${port} --target-port=8080
                """
                }
            }
        }
    }
}
