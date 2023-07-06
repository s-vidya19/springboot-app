pipeline {
    environment {
        registry = "cvaa/petclinic2"
        registryCredential = "DOCK_CRED_CVA"
        dockerImage = ''
        mavenHome = tool name: "Maven3", type: "maven"
        mavenCMD = "${mavenHome}/bin/mvn"
    }
    agent any 

    stages {
        stage('git clone') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'GIT_CRED', url: 'https://github.com/Cvaaaa/spring-petclinic-docker.git']])
                sh "${mavenCMD} clean install"
            }
        }
        
        stage('code quality check') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar') {
                       sh "${mavenCMD} sonar:sonar"
                    }
                }
            }
        }
        
        stage('upload to nexus') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'spring-petclinic', classifier: '', file: 'target/spring-petclinic-2.7.0-SNAPSHOT.jar', type: 'jar']], credentialsId: 'NEXUS_CRED', groupId: 'org.springframework.samples', nexusUrl: '54.236.7.176:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'aaa_aaa', version: '2.7.0-SNAPSHOT'
            }
        }
        
        stage('Build app') {
            steps {
                script{
                    dockerImage = docker.build registry + ":$BUILD_NUMBER"
                }
            }
        }
        
        stage('Deploy Image') {
            steps{
                script {
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }
        
        stage('Remove Unused docker image') {
            steps{
                sh "docker rmi $registry:$BUILD_NUMBER"
                sh "docker run -d --name petclinic123 -p 8082:8080 $registry:$BUILD_NUMBER"
            }
        }
    }
}
