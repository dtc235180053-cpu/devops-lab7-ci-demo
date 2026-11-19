pipeline {
    agent any

    tools {
        jdk 'JDK21'
        maven 'M3'
    }

    environment {
        SONAR_HOST_URL = 'http://lab7-sonarqube:9000'
        NEXUS_URL      = 'http://lab7-nexus:8081'
        NEXUS_REPO     = 'maven-snapshots'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'STEP 1/6: CHECKOUT — Lấy code từ GitHub...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'STEP 2/6: BUILD — Biên dịch với Maven...'
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                echo 'STEP 3/6: TEST — Chạy Unit Tests...'
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'STEP 4/6: SONARQUBE — Phân tích chất lượng code...'
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=sample-java-app \
                          -Dsonar.projectName="Sample Java App - Lab 7"
                    '''
                }
            }
        }

        stage('Package') {
            steps {
                echo 'STEP 5/6: PACKAGE — Đóng gói JAR...'
                sh 'mvn package -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }

        stage('Publish to Nexus') {
            steps {
                echo 'STEP 6/6: PUBLISH — Upload artifact lên Nexus...'
                withCredentials([usernamePassword(
                    credentialsId: 'nexus-credentials',
                    usernameVariable: 'NEXUS_USERNAME',
                    passwordVariable: 'NEXUS_PASSWORD'
                )]) {
                    configFileProvider([configFile(
                        fileId: 'maven-settings-nexus',
                        variable: 'MAVEN_SETTINGS'
                    )]) {
                        sh '''
                            mvn deploy -DskipTests \
                              -s $MAVEN_SETTINGS \
                              -DaltDeploymentRepository=nexus::default::${NEXUS_URL}/repository/${NEXUS_REPO}/
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline finished at: ${currentBuild.durationString}"
            cleanWs()
        }
        success {
            echo 'PIPELINE SUCCESS — Toàn bộ 6 stages hoàn thành!'
        }
        failure {
            echo 'PIPELINE FAILED — Kiểm tra logs để debug!'
        }
    }
}