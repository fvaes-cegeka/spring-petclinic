pipeline {
    agent any

    stages {
        stage('Debug docker') {
            steps {
                sh 'echo $PATH'
                sh 'which docker'
                sh 'docker --version'
            }
        }

        stage('Build and Test 🛠️') {
            steps {
                script {
                    def tag = createTag("${env.BRANCH_NAME}", "${env.BUILD_NUMBER}")
                    currentBuild.displayName = "Build #${env.BUILD_NUMBER} - ${tag}"
                    sh './mvnw --batch-mode --no-transfer-progress -e -U clean install -DskipTests=true -T 1'
                    sh './mvnw --batch-mode --no-transfer-progress -e -U verify -Pjacoco -T 1'
                }

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
                    def tag = createTag("${env.BRANCH_NAME}", "${env.BUILD_NUMBER}")
                    def imageName = "${namespace}:${tag}"

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

def createTag(String branchName, String buildNumber) {
    def pom = readMavenPom(file: 'pom.xml')
    def version = pom.version
    if (branchName == 'main') {
        return version
    } else {
        def branchSuffix = branchName.contains('/') ? branchName.tokenize('/').last() : branchName
        return "${version}.${branchSuffix}.alpha.${buildNumber}"
    }
}
