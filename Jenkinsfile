// Thsi Jenkins file is for Eureka Deployment

pipeline {
     agent {
          label 'k8s-slave'
     }
     tools {
        maven 'Maven-3.8.8'
        jdk 'JDK-17'
     }
     
     parameters {
        choice(name: 'scanOnly',
            choices: 'no\nyes',
            description: "This will scan the application"
        )
        choice(name: 'buildOnly',
            choices: 'no\nyes',
            description: 'This will Only build the application'
        )
        choice(name: 'dockerPush',
            choices: 'no\nyes',
            description: 'This will trigger the app build, docker build and docker push'
        )
        choice(name: 'deployToDev',
            choices: 'no\nyes',
            description: 'This will Deploy the application to Dev env'
        )
        choice(name: 'deployToTest',
            choices: 'no\nyes',
            description: 'This will Deploy the application to Test env'
        )
        choice(name: 'deployToStage',
            choices: 'no\nyes',
            description: 'This will Deploy the application to Stage env'
        )
        choice(name: 'deployToProd',
            choices: 'no\nyes',
            description: 'This will Deploy the application to Prod env'
        )
    }         
     environment {
         APPLICATION_NAME = "eureka"
         POM_VERSION = readMavenPom().getVersion()
         POM_PACKAGING = readMavenPom().getPackaging()
         DOCKER_HUB = "docker.io/kishoresamala84"
         DOCKER_CREDS = credentials('kishoresamala_docker_creds') 
     }

     stages {
        stage ('build'){
            when {
                anyOf {
                    expression {
                        params.buildOnly == 'yes'
                        params.dockerPush == 'yes'
                    }
                }
            }
             // This is where Build for Eureka application happens
             steps {
                 echo "Building ${env.APPLICATION_NAME} Application"
                 sh 'mvn clean package -DskipTest=true'
                 archiveArtifacts 'target/*.jar'
             }
        }

        stage ('sonarqube'){
            when {
                anyOf {
                    expression {
                        params.scanOnly == 'yes'
                        params.buildOnly == 'yes'
                        params.dockerPush == 'yes'
                    }
                }
            }
           steps {
               echo "*******Starting Sonar Scans with Quality Gates*********"
               withSonarQubeEnv('sonarqube') {// SonarQube is the name we configured in Manage Jenkins > system > Sonarqube , it hsould match exactly
                   sh """
                      mvn sonar:sonar \
                        -Dsonar.projectKey=i27-eureka \
                        -Dsonar.host.url=http://34.86.131.81:9000 \
                        -Dsonar.login=sqa_a584d652a3aa8d42bb12ce2763d16021898ebe63
                      """
                 }
                   timeout (time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
               }
             
           }
        } 

        stage ('BuildFormat') {
            steps {
               script { 
                   // Existing : i27-eureka-0.0.1-SNAPSHOT.jar
                   // Destination : i27-eureka-buildnumber-branchname.package
                 sh """
                   echo "Testing JAR Source: i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING}"
                   echo "Testing JAr Destination Format: i27-${env.APPLICATION_NAME}-${currentBuild.number}-${BRANCH_NAME}.${env.POM_PACKAGING}"
                 """
               }
            }
        }

        stage('Docker build and push') {
            when {
                anyOf{
                    expression {
                        params.buildOnly == 'yes'
                        params.dockerPush == 'yes'                        
                    }
                }
            }
            steps {
                script {
                    dockerbuildpush().call()                
                }
            }
        } 

        stage ('Deploy to Dev-env') {
            when {
                expression  {
                    params.deployToDev == 'yes'
                }
            }
            steps{  
                script {
                    dockerDeploy('dev','2761','8761').call()
                }          

             }  
           }

        stage ('Deploy to test-env') {
            when {
                expression  {
                    params.deployToTest == 'yes'
                }
            }
            steps {
                script {
                    dockerDeploy('test','6761','8761').call()
                }
            }
        }

        stage ("Deploy to Stage-env") {
            when {
                expression  {
                    params.deployToStage == 'yes'
                }
            }
            steps {
                script {
                    dockerDeploy('stage','7761','8761').call()
                }
            }
        }  

        stage ('Deploy to Prod-env') {
            when {
                expression  {
                    params.deployToProd == 'yes'
                }
            }
            steps {
                script {
                    dockerDeploy('prod','9761','8761').call()
                }
            }
        }
    }
}
    
def dockerbuildpush() {
    return {
      echo "****** Building Docker image *******"
      sh "cp ${WORKSPACE}/target/i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} ./.cicd"
      sh "docker build --no-cache --build-arg JAR_SOURCE=i27-${env.APPLICATION_NAME}-${env.POM_VERSION}.${env.POM_PACKAGING} -t ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} ./.cicd/"
      sh "docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}"
      echo "********* Push Image to Docker regitry ***********"
      sh "docker push ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT}"                
    }
}    

def dockerDeploy(envDeploy,hostPort,contPort) {
    return {
                echo "********* Deploying to dev Environment **************"
                withCredentials([usernamePassword(credentialsId: 'john_docker_vm_pwd', passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
                        script {
                            try {
                                sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@$dev_ip \"docker stop ${env.APPLICATION_NAME}-$envDeploy \""
                                sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@$dev_ip \"docker rm ${env.APPLICATION_NAME}-$envDeploy \""
                            }
                            catch(err){
                                echo "Error Caught: $err"
                            }
                            sh "sshpass -p '$PASSWORD' -v ssh -o StrictHostKeyChecking=no '$USERNAME'@$dev_ip \"docker container run -dit -p  $hostPort:$contPort --name ${env.APPLICATION_NAME}-$envDeploy ${env.DOCKER_HUB}/${env.APPLICATION_NAME}:${GIT_COMMIT} \""
                        }
                    }         
    }
}
