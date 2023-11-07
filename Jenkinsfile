pipeline {
    agent any
	 tools {
        jdk 'jdk'
        nodejs 'node'
    }
	
	 environment {
        	SCANNER_HOME = tool 'sonar'
		IMAGE = 'inderharrysingh/react-app'
    	}
	
    stages {
        stage('testing') {
            steps {
                script {
			cleanWs()
			sh 'git clone https://github.com/inderharrysingh/devops-project'
			sh 'echo "Starting the testing step...."'
                }
            }
}
	    
	stage('testing-sonar'){

		
		steps{
			withSonarQubeEnv('sonar') {
                    	sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=react\
                    	-Dsonar.projectKey=react'''
                	}		
       		}	
	}
	stage("quality gate") {
          	  steps {
                	script {
                   		 waitForQualityGate abortPipeline: false, credentialsId: 'sonar'
                }
            }
        }
	    
	    stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'checker'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
	    
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
	        
        stage('build') {
            	steps {
                	sh 'npm cache clean --force'
                	sh 'npm ci'
        	    }
	        }
	

	stage('push'){
		steps{
	  	  script {
			withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_HUB_USERNAME', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                        	sh "docker login -u $DOCKER_HUB_USERNAME -p $DOCKER_HUB_PASSWORD"
				 def TAG = sh(script: './update_tag', returnStdout: true).trim()
				sh "docker build -t $IMAGE:v$TAG ."
				sh "docker run -p 5000:80  $IMAGE:v$TAG "
				sh "docker push $IMAGE:v$TAG"
				
                    }
		}
         }

     }
}
}
