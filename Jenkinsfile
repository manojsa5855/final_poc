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
              git branch: 'vp-rem', credentialsId: 'gitcred', url: 'https://github.com/manojsa5855/vprofile-project.git'
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
              sh 'docker build -t manojsai5855/javapoc:$BUILD_NUMBER .'
       }
    }
    stage('Build Db Image'){
	    steps{
		    sh 'docker build -t manojsai5855/javapocdb:$BUILD_NUMBER -f Dockerfile1 .'
    }
    }
        stage('login') {
          steps{
              sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u manojsai5855 --password-stdin '
          }
        }
        stage('push') {
          steps{
          sh 'docker push manojsai5855/javapoc:$BUILD_NUMBER'
          sh 'docker push manojsai5855/javapocdb:$BUILD_NUMBER'

          }
	}
/*
       
        stage('Fetching cluster Infrastucture Code') {
          steps{
              git branch: 'main', url:'https://github.com/sharonraju143/Project_final.git'
          }  
        }
       
       stage('Terraform Init') {
            steps {
                script {
                    sh 'terraform init'
                    sh 'terraform apply --auto-approve -lock=false'
                    sh 'echo "[Master]" > /etc/ansible/hosts | az vm show -d -g Final_POC -n master-vm --query publicIps -o tsv >> /etc/ansible/hosts'
                    sh 'echo "[worker]" >> /etc/ansible/hosts | az vm show -d -g Final_POC -n worker-vm --query publicIps -o tsv >> /etc/ansible/hosts'
                    sh 'ansible-playbook site.yml'

                }
            }
       }
 */
       
/*       
       stage('Fetching k8s manifest files') {
          steps{
              git branch: 'main', url:'https://github.com/sharonraju143/kubernetes-poc.git'
          }  
        }
        
        stage('Deploying manifest files') {
            steps {
                script {
                    sh 'ansible-playbook deploy.yml'
                    
                }
            }
        }
*/
	   

    }
}

