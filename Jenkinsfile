@Library('my-shared-library') _

pipeline{

    agent any
    //agent { label 'Demo' }

    parameters{

        choice(name: 'action', choices: 'create\ndelete', description: 'Choose create/Destroy')
        string(name: 'ImageName', description: "name of the docker build", defaultValue: 'javapp')
        string(name: 'ImageTag', description: "tag of the docker build", defaultValue: 'v1')
        string(name: 'DockerHubUser', description: "name of the Application", defaultValue: 'manikanta513')
    }

    stages{
         
        stage('Git Checkout'){
                    when { expression {  params.action == 'create' } }
            steps{
            gitCheckout(
                branch: "main",
                url: "https://github.com/praveen1994dec/Java_app_3.0.git"
            )
            }
        }
         stage('Unit Test maven'){
         
         when { expression {  params.action == 'create' } }

            steps{
               script{
                   
                   mvnTest()
               }
            }
        }
         stage('Integration Test maven'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   mvnIntegrationTest()
               }
            }
        }
       // stage('Static code analysis: Sonarqube'){
       //  when { expression {  params.action == 'create' } }
       //     steps{
       //       script{
       //            
       //            def SonarQubecredentialsId = 'sonarqube-api'
       //            statiCodeAnalysis(SonarQubecredentialsId)
       //        }
       //     }
       //}
       stage ('Sonar qube code analaysis'){
            steps{
                withSonarQubeEnv('sonar-api'){
                    sh '/opt/apache-maven-3.6.3/bin/mvn clean package sonar:sonar'
                }
            }
       }
       stage('Quality Gate Status Check : Sonarqube'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   def SonarQubecredentialsId = 'sonarqube-api'
                   QualityGateStatus(SonarQubecredentialsId)
               }
            }
       }
        stage('Maven Build : maven'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   // mvnBuild()
                   sh '/opt/apache-maven-3.6.3/bin/mvn clean install -DskipTests'
               }
            }
        }
        //stage('Upload jar file to jfrog artifactory') {
        //    steps {
        //        script {
        //            def server = Artifactory.server 'jfrog'
        //            def uploadSpec = """{
        //                "files": [
        //                    {
        //                        "pattern": "target/kubernetes-configmap-reload-0.0.1-SNAPSHOT.jar",
        //                        "target": "example-repo-local"
        //                    }
        //                ]
        //            }"""
        //            server.upload(uploadSpec)
        //        }
        //    }
        //}

        stage('Upload to Artifactory') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'jfrog-cred', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh """
                        curl -u${USERNAME}:${PASSWORD} -T target/kubernetes-configmap-reload-0.0.1-SNAPSHOT.jar "http://192.168.33.10:8082/artifactory/example-repo-local/"
                        """
                    }
                }
            }
        }
        stage('Docker Image Build'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   dockerBuild("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
               }
            }
        }
         stage('Docker Image Scan: trivy '){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   dockerImageScan("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
               }
            }
        }
        stage('Docker Image Push : DockerHub '){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   dockerImagePush("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
               }
            }
        }   
        stage('Docker Image Cleanup : DockerHub '){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   dockerImageCleanup("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
               }
            }
        }      
    }
}
