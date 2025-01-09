pipeline {
    agent any
    tools{
        maven 'Maven3'
    }
    environment {
        registry = '261884385716.dkr.ecr.us-west-1.amazonaws.com/abb-docker-repo'
        registryCredential = 'jenkins-ecr-login-credentials'
        dockerimage = ''
    }
    stages{
        stage("Checkout the project") {
           steps{
               git branch: 'master', url: 'https://github.com/NandishDevops-27/sonar-springboot-app.git'
           } 
        }
        stage("Build the package"){
            steps {
                sh 'mvn clean install'
            }
        }
		stage("Sonar Quality Check"){
		    steps{
		        script{
		            withSonarQubeEnv(installationName: 'sonar-', credentialsId: 'jenkins-sonar-token') {
		                sh 'mvn sonar:sonar'
	    	        }
	    	            timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                            if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
	    	        }
		        }
            }
        }	
        stage('Building the Image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        stage ('Deploy the Image to Amazon ECR') {
            steps {
                script {
                    docker.withRegistry("http://" + registry, "ecr:us-west-1:" + registryCredential ) {
                    dockerImage.push()
                    }
                }
            }
        }
    }
}