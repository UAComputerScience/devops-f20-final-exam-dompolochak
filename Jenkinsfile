pipeline {
    agent{
        docker
        {
            image 'p2ubuntu2004'
            args '--user root'
        }
    }

    stages {
        stage('Setup') {
            steps {
                sh 'cat /etc/os-release'
                sh 'rm -rf devops-f20-final-exam-dompolochak'
                sh 'git clone https://github.com/UAComputerScience/devops-f20-final-exam-dompolochak.git'
                sh 'cd devops-f20-final-exam-dompolochak'
            }
           
        }
        stage('CMake'){
            steps{
                sh 'mkdir build'
                sh 'cd build; pwd; cmake ../devops-f20-final-exam-dompolochak/markturn -G Ninja'
            }
        }
        stage('Build'){
            steps{
                sh 'cd build; ninja'
            }
        }
        stage('Test')
        {
            steps{
                sh 'cd build; ctest'
            }
        }
        stage('Package')
        {
            steps{
                sh 'cd build; cpack'
            }
        }
    }
}