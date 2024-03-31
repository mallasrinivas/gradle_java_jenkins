pipeline {
    agent any
    tools {
        jdk 'jdk11'
        gradle 'gradle'
    }
    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        stage('git clone') {
            steps {
                git branch: 'main', url: 'https://github.com/mallasrinivas/gradle_java_jenkins.git'
            }
        }
        stage('gradle compline') {
            steps {
                sh 'chmod +x gradlew'
                sh './gradlew compileJava'
            }
        }
        stage('test Gradle') {
            steps {
                sh 'chmod +x gradlew'
                sh './gradlew test'
            }
        }
        stage('sonar analysis') {
            steps {
                script {
                    /* groovylint-disable-next-line NestedBlockDepth */
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'chmod +x gradlew'
                        sh '/var/lib/jenkins/workspace/Gradle-demo/gradlew sonarqube'
                    }
                //    timeout( time: 10, unit: 'MINUTES' ) {
                //       def qg = waitForQualityGate()//waitForQualityGate
                //       if (qg.status != 'OK') {
                //          error "Pipeline aborted due to quality gate failure: ${qg.status}"
                //       }
                //    }
                }
            }
        }
        stage('build project') {
            steps {
                sh 'chmod +x gradlew'
                sh './gradlew build'
            }
        }
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.html'
            }
        }
        stage('build and push to nexus') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                        sh '''
                          docker build -t 13.201.12.82:8083/gradle1:latest .
                          docker login -u admin -p $docker_password 13.201.12.82:8083
                          docker push 13.201.12.82:8083/gradle1:latest
                         '''
                    }
                }
            }
        }
        stage('deploy to container') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                        sh 'docker run -d --name g1 -p 8082:8080 13.201.12.82:8083/gradle1:latest'
                    }
                }
            }
        }
    }
}
