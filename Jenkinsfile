def COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger,'
    
]
pipeline {
    agent any 
    tools {
        maven "MAVEN3"
        jdk "OracleJDK11"
    }
    stages {
        stage('Fetch Code'){
            steps {
                git branch: 'main', url: 'https://github.com/devopshydclub/vprofile-project.git'
        }
    }
        stage('Build') {
            steps {
                sh 'mvn install -DskipTests'
            }

            post{
                success{
                    echo "Archiving artifacts now..."
                    archiveArtifacts artifacts: '**/*.war'
            }
        }
    }
        stage('UNIT TEST') {
            steps {
                sh 'mvn test'
            }
        }

        stage("Checkstyle Analysis") {
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar4.8'
            }
            steps{
                withSonarQubeEnv('sonar') {
                      sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }

    }
        stage("Upload t Artifact to Nexus ") {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '172.31.21.181:8081',
                    groupId: 'QA',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: 'vprofile-repo',
                    credentialsId: 'nexuslogin',
                    artifacts: [
                     [artifactId: 'vproapp',
                        classifier: '',
                        file: 'target/vprofile-v2.war',
                        type: 'war']
        ]
     )
            }
        }}
            post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}