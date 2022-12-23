pipeline{
    agent{
        node{
            label 'awsec2instance'
        }
    }
    tools{
        maven 'MAVEN'
    }
    stages{
        stage('git checkout'){
            steps{
                script{
                    git branch: 'main', url: 'https://github.com/Chanti369/fullproject.git'
                }
            }
        }
        stage('mvn clean install'){
            steps{
                script{
                    sh 'mvn clean install'
                }
            }
        }
        stage('static code analysis'){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonarqube-token') {
                        sh 'mvn clean install sonar:sonar'
                    }
                }
            }
        }
        stage('Quality gate'){
            steps{
                script{
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-token'
                }
            }
        }
        stage('nexus artifact uploader'){
            steps{
                script{
                    def readpom = readMavenPom file: 'pom.xml'
                    def readversion = readpom.version
                    def readrepo = readversion.endsWith("SNAPSHOT") ? "fullproject-snapshot" : "fullproject-release"
                    nexusArtifactUploader artifacts: [[artifactId: 'devops-integration', classifier: '', file: 'target/devops-integration.jar', type: 'jar']], credentialsId: 'nexus', groupId: 'com.javatechie', nexusUrl: '3.109.5.142:8081', nexusVersion: 'nexus3', protocol: 'http', repository: readrepo, version: readversion
                }
            }
        }
        stage('docker build image'){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', passwordVariable: 'dockerpassword', usernameVariable: 'dockerusername')]) {
                        sh 'docker login -u $dockerusername -p $dockerpassword'
                        sh 'docker build -t $JOB_NAME:v1.$BUILD_ID .'
                        sh 'docker tag $JOB_NAME:v1.$BUILD_ID $dockerusername/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker tag $JOB_NAME:v1.$BUILD_ID $dockerusername/$JOB_NAME:latest'
                        sh 'docker rmi -f $JOB_NAME:v1.$BUILD_ID'
                    }
                }
            }
        }
    }
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "adityakumarreddyvennapusa@gmail.com";  
		}
	}
}