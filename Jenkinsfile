def COLOR_MAP=[
    'SUCCESS':'good',
    'FAILURE':'danger'
]

pipeline{
    agent any
    tools {
        maven 'maven'
        jdk 'JDK'
    }
    environment{
    SNAPREPO = 'Vpro-snapshots'
    NEXUSUSER = 'admin'
    nexuspassword = 'admin'
    releaserepo = 'vpro-release'
    centralrepo = 'Vpro-mavan-central'
    nexusip = '172.31.30.167'
    nexusport = '8081'
    nexusgroup = 'Vpro-maven-group'
    nexuslogin = 'nexuslogin'
    SONARSERVER = 'Sonarserver'
    SONAR_SCANNER = 'Sonarscanner'
    }
    stages{
        stage('BUILD'){
            steps{
                sh 'mvn -s settings.xml install -DskipTests'
            }
            post {
                success {
                    echo 'Now archiving'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('UNIT TEST'){
            steps{
                sh 'mvn -s settings.xml test'
            }
            post {
                success {
                    slackSend channel: '#devops-project',
                    color: 'good',
                    message: "UNIT TEST IS SUCCESS"
                }
                failure {
                    slackSend channel: '#devops-project',
                    color: 'danger',
                    message: "UNIT TEST IS FAILED"
                }
            }
        }
        stage('INTEGRATION TEST'){
            steps{
                sh 'mvn -s settings.xml verify -DskipUnitTests'
            }
        }
        stage('CHECKSTYLE ANALYSIS'){
            steps{
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }
        stage('UPLOAD ARTIFACT TO NEXUS'){
            steps{
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${nexusip}:${nexusport}",
                    groupId: 'SALES',
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    repository: "${releaserepo}",
                    credentialsId: "${nexuslogin}",
                    artifacts: [
                        [artifactId: 'vproapp',
                        classifier: '',
                        file: 'target/vprofile-v2.war',
                        type: 'war']
                    ]
                )
            }
        }
    }
    post {
        always {
            echo 'slack notifications'
            slackSend channel: '#project',
            color: COLOR_MAP[currentBuild.currentResult],
            message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} time ${env.BUILD_TIMESTAMP} \n More info at: ${BUILD_URL}"
        }
    }
}
