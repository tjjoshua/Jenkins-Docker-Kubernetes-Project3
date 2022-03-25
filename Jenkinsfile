pipeline {
    agent any
	tools {
		//maven 'Maven'
		maven 'Maven-3.6.1'
	}
	
	environment {
		PROJECT_ID = 'sage-sunrise-344921'
                CLUSTER_NAME = 'gke-cluster'
                LOCATION = 'us-central1-c'
                CREDENTIALS_ID = 'k8s'		
	}
	
    stages {
	    stage('Scm Checkout') {
		    steps {
			    checkout scm
		    }
	    }
	    
	    stage('Build') {
		    steps {
			    sh 'mvn clean package'
		    }
	    }
	    
	    stage('Test') {
		    steps {
			    echo "Testing..."
			    sh 'mvn test'
		    }
	    }
	    
	    stage('Build Docker Image') {
		    steps {
			    sh 'whoami'
			    script {
				    myimage = docker.build("tjjoshua/devops:${env.BUILD_ID}")
			    }
		    }
	    }
	    
	    stage("Push Docker Image") {
		    steps {
			    script {
				    echo "Push Docker Image"
				    // withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerhub')])
				    withCredentials([usernamePassword(credentialsId: 'GitHub-Creds', passwordVariable: 'passwd', usernameVariable: 'username')]) {
            				//sh "docker login -u ameintu -p ${dockerhub}"
					    sh "docker login -u ${username} -p ${passwd}"
				    }
				        myimage.push("${env.BUILD_ID}")
				    
			    }
		    }
	    }
	    
	    stage('Deploy to K8s') {
		    steps{
			    echo "Deployment started ..."
			    sh 'ls -ltr'
			    sh 'pwd'
			    sh "sed -i 's/tagversion/${env.BUILD_ID}/g' serviceLB.yaml"
				sh "sed -i 's/tagversion/${env.BUILD_ID}/g' deployment.yaml"
			    echo "Start deployment of serviceLB.yaml"
			    sh "whoami"
			    step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'serviceLB.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
			    echo "Start deployment of deployment.yaml"
			    //sh "whoami"
			    //step([projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, credentialsId: env.CREDENTIALS_ID])
			    //sh "whoami"	
			    step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
			    echo "Deployment Finished ..."
		    }
	    }
    }
}
