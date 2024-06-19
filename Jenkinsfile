pipeline {
    agent any

    environment {
        CC = 'g++'
        CXX = 'g++'
        GCOV = 'gcov'
        SONAR_SCANNER = '/usr/local/bin/sonar-scanner'
        SONARQUBE_URL = 'http://sonarqube:9000'
        SONARQUBE_LOGIN = 'your_sonarqube_token'  // Replace with your actual SonarQube token
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://your-git-repo-url.git'
            }
        }
        
        stage('Build') {
            steps {
                sh '''
                    mkdir -p build
                    cd build
                    cmake ..
                    make
                '''
            }
        }
        
        stage('Unit Tests') {
            steps {
                sh '''
                    cd build
                    ctest --output-on-failure
                '''
            }
        }
        
        stage('Code Coverage') {
            steps {
                sh '''
                    cd build
                    gcovr --xml --output coverage.xml
                '''
                publishCoverage adapters: [gcovAdapter('build/coverage.xml')]
            }
        }

        stage('Static Code Analysis') {
            steps {
                sh '''
                    cppcheck --enable=all --xml --xml-version=2 . 2> cppcheck.xml
                '''
                recordIssues tools: [cppCheck(pattern: 'cppcheck.xml')]
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') { 
                    sh "${SONAR_SCANNER} -Dsonar.projectKey=your_project_key -Dsonar.sources=. -Dsonar.host.url=${SONARQUBE_URL} -Dsonar.login=${SONARQUBE_LOGIN}"
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        waitForQualityGate abortPipeline: true
                    }
                }
            }
        }
    }

    post {
        always {
            junit 'build/test-results/*.xml'
            cleanWs()
        }
    }
}
