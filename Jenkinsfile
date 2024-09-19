pipeline {
    agent any

    environment {
        MAVEN_HOME = tool name: 'M3', type: 'maven' // Adjust based on your Maven setup in Jenkins
        SONARQUBE_URL = 'http://your-sonarqube-server-url'
        ARTIFACTORY_SERVER = 'your-artifactory-server-id'
        TOMCAT_URL = 'http://your-tomcat-server-url/manager/text/deploy?path=/your-app&update=true'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out the code from Git...'
                git branch: 'main', url: 'https://github.com/your-username/maven-tomcat-deployment.git'
            }
        }

        stage('Build with Maven') {
            steps {
                echo 'Building the project using Maven...'
                withMaven(maven: 'M3') {
                    sh 'mvn clean install'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    mvn sonar:sonar \
                    -Dsonar.projectKey=sample-webapp \
                    -Dsonar.host.url=${SONARQUBE_URL} \
                    -Dsonar.login=${SONARQUBE_CRED}
                    '''
                }
            }
        }

        stage('Upload to JFrog Artifactory') {
            steps {
                echo 'Uploading artifact to JFrog Artifactory...'
                script {
                    def server = Artifactory.server(ARTIFACTORY_SERVER)
                    def uploadSpec = """{
                        "files": [{
                            "pattern": "target/*.war",
                            "target": "your-repository/your-path/"
                        }]
                    }"""
                    server.upload(uploadSpec)
                }
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                echo 'Deploying WAR file to Tomcat...'
                sh '''
                curl --upload-file target/sample-webapp.war \
                     -u ${TOMCAT_CRED} \
                     "${TOMCAT_URL}"
                '''
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed'
        }
    }
}
