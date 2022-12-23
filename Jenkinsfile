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
        stage('docker push image'){
            steps{
                script{
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', passwordVariable: 'dockerpassword', usernameVariable: 'dockerusername')]) {
                        sh 'docker login -u $dockerusername -p $dockerpassword'
                        sh 'docker push $dockerusername/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker push $dockerusername/$JOB_NAME:latest'
                        sh 'docker rmi -f $dockerusername/$JOB_NAME:v1.$BUILD_ID'
                        sh 'docker rmi -f $dockerusername/$JOB_NAME:latest'
                    }
                }
            }
        }
        stage('ansible-playbook'){
            steps{
                script{
                    sh 'ansible-playbook ansible.yml'
                }
            }
        }
        stage('helm datree'){
            steps{
                script{
                    dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=6bc37af1-e6bb-41e2-886b-ee0adad572d7']) {
                             sh 'helm datree test myapp'
                        }
                    }
                }
            }
        }
        stage('pushing helm chart to nexus repo'){
            steps{
                script{
                    dir('kubernetes/') {
                        withCredentials([usernamePassword(credentialsId: 'nexus-creds', passwordVariable: 'nexuspassword', usernameVariable: 'nexususername')]) {
                            sh '''
                            helmversion=$(helm show chart myapp | grep version | cut -d ":" -f 2 | tr -d " ")
                            tar -czvf myapp-${helmversion}.tgz myapp/
                            curl -u $nexususername:$nexuspassword http://3.109.5.142:8081/repository/helm-repo/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                        }
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