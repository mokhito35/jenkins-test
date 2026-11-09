pipeline {
    agent {
        label 'Second'
    }

    stages {
        stage('Hello') {
            steps {
                echo 'Hello from Jenkins!'
            }
        }

        stage('Check Environment') {
            steps {
                bat 'whoami'
                bat 'java -version'
                bat 'git --version'
            }
        }

        stage('Build') {
            steps {
                echo 'Build completed successfully!'
            }
        }
        stage('Check Git') {
    steps {
        bat 'git status'
        bat 'git branch'
    }
        } 
    }
}
