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
    
    

![image](https://user-images.githubusercontent.com/130491596/234239066-c0e986e6-3299-4fbe-b598-0a03bee50639.png)


![image](https://user-images.githubusercontent.com/130491596/234239337-5e6560e2-68ff-47af-96a6-78f011db0a09.png)

![image](https://user-images.githubusercontent.com/130491596/234239484-08e03e06-5963-4d9b-ab48-ee37265e741b.png)


![image](https://user-images.githubusercontent.com/130491596/234239821-ce61291d-6b58-49df-b61d-e0253723edb1.png)


![image](https://user-images.githubusercontent.com/130491596/234240409-6e478c4c-697e-47e3-9fd4-1040e886220d.png)




    
    
    
