pipeline {
    agent any 

    tools{
        maven 'Maven 3.9.4'
    }

    triggers {
        pollSCM "* * * * *"
    }

    options {
        timestamps ()
        ansiColor("xterm")
    }

    parameters {
        boleanParam(
            name: "RELEASE",
            description: "Build a release from current commit.",
            defaultValue: false)
    }

    environment {
        dockerimagename = "pancaaa/springboot-app"
        dockerImage = ""
        registryCredential = 'dockerhublogin'

        gitUrl = 'https://github.com/war3wolf/mvn-springboot.git'
        gitBranch = 'development'
    }

    stages {
        stage('Checkout Source') {
            steps {
                sh 'rm -rf *'
                git branch: gitBranch,
                credentialsId: 'Github Connection',
                url: gitUrl
            }
        }
        
        stage ('Build & Deploy SNAPSHOT') {
            steps {
                
                sh "mvn -B deploy"

                // sh 'mvn package spring-boot:repackage'
                // sh 'mvn clean install -DskipTests'
            }
        }

        stage ('Release') {
            when {
                allof {
                    sh 'rm -rf *'
                    git branch: gitBranch,
                    credentialsId: 'Github Connection',
                    url: gitUrl
                    expression { param.RELEASE }
                }
                    
            }
            steps {
                sh "mvn -B release:prepare"
                sh "mvn -B release:perform"
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    dockerImage = docker.build dockerimagename
                }
            }
        }
	    stage('Pushing Image') {
            steps{
                script {
                    docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
                    dockerImage.push("latest")
                    }
                }   
            }
        }

        stage('Deploy to Kube Cluster  '){
            steps{
                script{
                    sh 'kubectl --kubeconfig=/home/jenkins/.kube/dev-cluster/config config current-context'
                    sh 'kubectl --kubeconfig=/home/jenkins/.kube/dev-cluster/config delete deployment hello-world'
                    sh 'kubectl --kubeconfig=/home/jenkins/.kube/dev-cluster/config delete service hello-world'
                    sh 'kubectl --kubeconfig=/home/jenkins/.kube/dev-cluster/config apply -f /var/lib/jenkins/workspace/Demo_Deploy_master/deployment.yaml'
                    sh 'kubectl --kubeconfig=/home/jenkins/.kube/dev-cluster/config apply -f /var/lib/jenkins/workspace/Demo_Deploy_master/service.yaml'        
                }
            }
        }
    }
}