pipeline {
    agent any
    tools {
	    maven "MAVEN3"
	    jdk "OracleJDK8"
	}
	environment{
	    DOCKERHUB_CREDENTIALS=credentials('dockerhubcred')
	    //appRegistry = "manojsai5855/javapocdb"
	}
	
    stages{
        stage('Fetch code') {
          steps{
              git branch: 'main', credentialsId: 'gitcred', url: 'https://github.com/manojsa5855/final_poc.git'
          }  
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('Test'){
            steps {
                sh 'mvn test'
            }

        }


        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar4.8'
            }
            steps {
               withSonarQubeEnv('sonar') {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }


    stage('Build App Image') {
       steps {
              sh 'docker build -t manojsai5855/javafinalimg .'
       }
    }
    stage('Build Db Image'){
	    steps{
		    sh 'docker build -t manojsai5855/javafinaldb -f Dockerfile1 .'
    }
    }
        stage('login') {
          steps{
              sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u manojsai5855 --password-stdin '
          }
        }
        stage('push') {
          steps{
          sh 'docker push manojsai5855/javafinalimg'
          sh 'docker push manojsai5855/javafinaldb'

          }
	}
	stage('deploying helm chart'){
		steps{
			withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubeconfig', namespace: '', restrictKubeConfigAccess: true, serverUrl: 'https://12.0.0.4:6443') {
    sh 'helm upgrade --install firstrelease javapoc'
}
		}
	}


	   

    }
}

