pipeline {
    agent {
        label 'k8s-slave'
    }
    tools {
        maven 'Maven-3.8.8'
        jdk 'JDK-17'
    }
     environment {
        APPLICATION_NAME = "eureka"
     }
     stages {
        stage ('***** Build Stage *****') {
            steps {
                sh "mvn clean package -DskipTest=true"
                archiveArtifacts 'target/*.jar'
            }

        }
     }
}
