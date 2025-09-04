pipeline {

    agent {
        docker {
            image 'openjdk:17-jdk'
            args '-v /var/run/docker.sock:/var/run/docker.sock -v /tmp/jenkins-m2:/root/.m2'
        }
    }

    environment {
        PATH = "/usr/local/bin:${env.PATH}"
        QODANA_TOKEN=credentials('qodana-token')
        QODANA_ENDPOINT='https://qodana.cloud'
    }

    stages {
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
        stage('Quality Analysis with Qodana 🔍') {
            agent {
                docker {
                    args '''
                      -v "${WORKSPACE}":/data/project
                      -v /var/run/docker.sock:/var/run/docker.sock
                      --entrypoint=""
                      '''
                    image 'jetbrains/qodana-jvm-community:2025.1'
                }
            }
            steps {
                sh '''qodana --property=project.open.type=Maven'''
            }
        }
        stage('Create Docker image and push 🐳') {
            steps {
                script {
                    def imageName = "fvaescegeka/spring-petclinic"
                    def tag = createTag("${env.BRANCH_NAME}", "${env.BUILD_NUMBER}")
                    def image = "${imageName}:${tag}"

                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh "docker build -t ${image} ."
                        sh "echo \$DOCKERHUB_PASSWORD | docker login -u \$DOCKERHUB_USERNAME --password-stdin"
                        sh "docker push ${image}"
                    }
                }
            }
        }
        stage('Git tag 🏷️') {
            steps {
                script {
                    def tag = createTag("${env.BRANCH_NAME}", "${env.BUILD_NUMBER}")

                    sshagent(['github-credentials']) {
                        sh "git remote set-url origin git@github.com:fvaes-cegeka/spring-petclinic.git"
                        sh "git tag -a ${tag} -m 'Tag created by Jenkins for build ${tag}'"
                        sh "git push origin ${tag}"
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
