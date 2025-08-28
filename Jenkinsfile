pipeline {
    agent any

    stages {
        stage('Build and Test 🛠️') {
            steps {
                sh './mvnw --batch-mode --no-transfer-progress -e -U clean install -DskipTests=true -T 1'
                sh './mvnw --batch-mode --no-transfer-progress -e -U verify -Pjacoco -T 1'
            }

            post {
                always {
                    archiveArtifacts artifacts: '**/target/surefire-reports/*.xml', allowEmptyArchive: true
                    junit testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true
                }
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
