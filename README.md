# My-Repo


pipeline {
    agent any
     tools {
    maven 'Maven'
  }
    stages{
        stage('Checkout SCM') {
      steps {
          git url: 'https://github.com/smondepulanka/docker-hello-world-spring-boot.git',
          branch: 'master'
        }
      }
         stage('Artifactory-Configuration') {
            steps {
                rtMavenDeployer (
                    id: 'spc-deployer',
                    serverId: 'jfrog1',
                    releaseRepo: 'test-repo-libs-release-local',
                    snapshotRepo: 'test-repo-libs-snapshot-local',
                )
            }
        }
         stage('Build the Code and sonarqube-analysis') {
             steps {
                withSonarQubeEnv('sonarq') {
                    sh script: "mvn clean package sonar:sonar"
                }
            rtMavenRun (
                    // Tool name from Jenkins configuration.
                    tool: 'Maven',
                    pom: 'pom.xml',
                    goals: 'clean install',
                    // Maven options.
                    deployerId: 'spc-deployer',
                )

             }
         }
        stage('Build docker image'){
            steps{
                script{
                    sh """docker build -t smondepulanka/helloworld:${BUILD_NUMBER} ."""
                }
            }
        }
       stage('Push image to Hub'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'dockerhublogin', passwordVariable: 'DOCPASS', usernameVariable: 'DOCUSER')]) {
                  script{
                   sh 'docker login -u $DOCUSER -p $DOCPASS'
                   sh 'docker push smondepulanka/helloworld:${BUILD_NUMBER}'
                }
              }
            }
        }
        stage('Update docker image'){
            steps{
                script{
                    sh """
                    sed -i "s/1.0/${BUILD_NUMBER}/g" $WORKSPACE/deployment.yaml
                    """
                }
            }
        }        
       stage('Deploying App to Kubernetes') {
         steps {
           script {
              kubernetesDeploy(configs: "deployment.yaml", kubeconfigId: "kubernetes")
        }
      }
    }
  }
} 
    
    
    
    
    
    
