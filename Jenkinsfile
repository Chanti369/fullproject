pipeline{
    agent{
        node{
            label 'awsec2instance'
        }
    }
    stages{
        stage('git checkout'){
            steps{
                script{
                    git branch: 'main', url: 'https://github.com/Chanti369/fullproject.git'
                }
            }
        }
    }
}