pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/killerspeed/most_task'
            }
        }

        stage('Run Script') {
            steps {
                sh './hello.sh'
            }
        }

        stage('Save Artifact') {
            steps {
                // Сохраняем вывод в файл
                sh './hello.sh > output.txt'
                // Архивируем артефакт в Jenkins
                archiveArtifacts artifacts: 'output.txt', allowEmptyArchive: false
                // Копируем в произвольную директорию на сервере
                sh 'mkdir -p /tmp/ci_artifacts && cp output.txt /tmp/ci_artifacts/'
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed!'
        }
        success {
            echo 'Pipeline succeeded!'
        }
    }
}
