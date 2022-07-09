pipeline {
    
    environment {
        //registry = "https://docker.io/nitindadev/finance-app"
        dockerhub = credentials('dockerhub')
        dockerImage = ''
    }
    
    agent any

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "m3"
    }

    stages {
        stage('Code checkout') {
            steps{
            git credentialsId: 'git-cred-repo-priv', url: 'https://github.com/Nitinda97/project-on-aks.git'
            }
        }
        stage('Build the code') {
            steps{
            sh "mvn -f Financy/pom.xml -Dmaven.test.failure.ignore=true clean package"
            }
        }
        
        stage('Unit-Test'){
            
            steps{
            sh "mvn -f Financy/pom.xml test"
            }
            
            post {
                //if maven build was able to run the test we will create a test report and archive the jar in local machine
                success {
                    junit '**/target/surefire-reports/*.xml'
                    archiveArtifacts 'Financy/target/*.jar'
                }}
            
        }
        
           
        stage('Checkstyle') {
            steps{
            sh "mvn -f Financy/pom.xml checkstyle:checkstyle"
            }}
            
        stage('checkstyle Report') {
            steps {
                recordIssues(tools: [checkStyle(pattern: 'Financy/target/checkstyle-result.xml')])
            }
        }
        
        stage('Code Coverage') {
            steps {
                jacoco()
            }
        }
        
        
        stage('code quality check sonar') {
            
            steps {
                
                sh 'mvn -f Financy/pom.xml clean verify sonar:sonar -Dsonar.projectKey=finance-app -Dsonar.host.url=http://20.84.83.13:9000 -Dsonar.login=sqp_d4fb5046256cb469dd795e0c5905571af36062b0'
            }
            
        }
        
        
        
        stage ('Upload jar to Artifactory')  {
              
            
          steps {
          nexusArtifactUploader(
          nexusVersion: 'nexus3',
          protocol: 'http',
          nexusUrl: '20.115.61.38:8081',
          groupId: 'financy',
          version: '0.0.1-SNAPSHOT',
          repository: 'maven-snapshots',
          credentialsId: 'nexus-cred',
          artifacts: [
            [artifactId: 'financy',
             classifier: '',
             file: 'Financy/target/financy-1.0.10-SNAPSHOT.jar',
             type: 'jar']
        ]
        )
          }
        }
        
        
        
        stage('docker image') {
            
            
            steps {
                script {
                    dockerImage=docker.build("finance-app:${env.BUILD_NUMBER}","-f Financy/Dockerfile .")
                }
            }
        }
        
        stage('docker push on Dockerhub') {
            
            
            steps {
                
                sh "docker tag finance-app:${env.BUILD_NUMBER} nitindadev/finance-app:${env.BUILD_NUMBER}"
                
                sh 'echo $dockerhub_PSW | docker login -u $dockerhub_USR --password-stdin'
                
                sh "docker push nitindadev/finance-app:${env.BUILD_NUMBER}"
                
            }
        }
        
         stage ('Deploy on AKS cluster'){

            steps{
                
                withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'aks-secret', namespace: 'default', serverUrl: 'https://aksdemo1-dns-8dbf0ba8.hcp.eastus.azmk8s.io:443') {
                  sh "kubectl apply -f db-deployment"  
                  sh "export IMAGE_TAG=${env.BUILD_NUMBER}; envsubst < finance-app-deployment.yml | kubectl apply -f -"
                  sh "docker rmi nitindadev/finance-app:${env.BUILD_NUMBER}"
            }}
            }
        
        }
        
        
    }
