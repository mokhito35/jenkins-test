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
         stage('Check VM') {
            steps {
                echo 'Checking target VM...'
                bat 'hostname'
                bat 'whoami'
            }
        }

        stage('Restart VM') {
            steps {
                echo 'Scheduling Windows VM restart in 60 seconds...'
                bat 'shutdown /r /t 60 /c "Restart requested by Jenkins"'
            }
        }
    }
}