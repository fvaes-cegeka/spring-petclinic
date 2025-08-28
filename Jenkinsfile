pipeline {
    agent any

    script {
        def createDockerTag(String branchName) {
            def pom = readMavenPom(file: 'pom.xml')
            def version = pom.version
            if (branchName == 'main') {
                return version
            } else {
                def branchSuffix = branchName.contains('/') ? branchName.tokenize('/').last() : branchName
                return "${version}.${branchSuffix}"
            }
        }
        env.TAG = createDockerTag(env.BRANCH_NAME)
    }

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
        stage('JaCoCo Report 📊') {
             steps {
                jacoco(
                    execPattern: '**/jacoco.exec',
                    classPattern: '**/classes',
                    sourcePattern: '**/src/main/java'
                )
             }
        }
        stage('Create Docker image and push 🐳') {
            steps {
                script {
                    def namespace = "fvaescegeka"
                    def imageName = "${namespace}:${env.TAG}"

                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh "docker build -t ${imageName} ."
                        sh "echo \$DOCKERHUB_PASSWORD | docker login -u \$DOCKERHUB_USERNAME --password-stdin"
                        sh "docker push ${imageName}"
                    }
                }
            }
        }
    }
}
