pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/killerspeed/most_task.git'
            }
        }

        // Добавляем этот этап!
        stage('Set Permissions') {
            steps {
                sh 'chmod +x hello.sh'
                sh 'ls -la hello.sh'  // Проверка прав (должно быть -rwxr-xr-x)
            }
        }

        stage('Run Script') {
            steps {
                sh './hello.sh'
            }
        }

        stage('Save Artifact') {
            steps {
                sh './hello.sh > output.txt'
                archiveArtifacts artifacts: 'output.txt', allowEmptyArchive: false
                sh 'mkdir -p /tmp/ci_artifacts && cp output.txt /tmp/ci_artifacts/'
            }
        }
    }
}
