pipeline {
     agent any
//     environment {
//     CI = true
//     ARTIFACTORY_ACCESS_TOKEN = credentials('artifactory-access-token')
//     JFROG_PASSWORD  = credentials('jfrog-password')
//   }
    stages {
        stage('Checkout Git Repository')
         {
            steps {
                sh 'rm -rf Non_Container_Deployment'
                sh 'git clone https://github.com/Rajanlt/Non_Container_Deployment.git'
            }
        }
        stage('Build Maven Job'){
           steps {
              sh 'mvn install'
           }
        }
       stage('Upload Binaries to Nexus Artifactory')
         {
        steps 
              {
        nexusArtifactUploader artifacts: [[artifactId: 'http://13.233.166.229:8081/repository/maven-snapshots/', classifier: '', file: '/workspace/target/*.war', type: 'maven-snapshots']], credentialsId: 'nexus_id', groupId: 'nexus', nexusUrl: '13.233.166.229:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'http://13.233.166.229:8081/repository/maven-snapshots/', version: 'nexusid'       
        }
         }
        stage('Building Docker Image'){
        steps{
              sh '''
              docker build -t localhost:8082/java-web-app-docker/demoapp:$BUILD_NUMBER --pull=true .
              docker images
              '''
          }
      }
      stage('Image Scanning Trivy'){
            steps{
               sh 'trivy image localhost:8082/java-web-app-docker/demoapp:$BUILD_NUMBER > $WORKSPACE/trivy-image-scan/trivy-image-scan-$BUILD_NUMBER.txt'
            }
     }
//      stage('Uploading Image Scan to Jrog Artifactory'){
//          steps{
//           sh 'jf rt upload --url http://172.17.0.3:8082/artifactory/ --access-token ${ARTIFACTORY_ACCESS_TOKEN} trivy-image-scan/trivy-image-scan-$BUILD_NUMBER.txt trivy-scan-files/'           
//          }
//      }
     stage('Pushing Docker Image into Jfrog'){
         steps{
             sh '''
             docker login java-web-app-docker.jfrog.io -u admin -p ${JFROG_PASSWORD}
             docker push localhost:8082/java-web-app-docker/demoapp:$BUILD_NUMBER
             '''
        }
     }
     stage('Cleaning up DockerImage'){
            steps{
                sh 'docker rmi localhost:8082/java-web-app-docker/demoapp:$BUILD_NUMBER'
           }
       }
  }
}
