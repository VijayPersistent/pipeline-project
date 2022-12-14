pipeline {
  agent any
  tools {
  
  maven 'maven-3.8.6'
   
  }
    stages {

      stage ('Checkout SCM'){
        steps {
          git branch: 'main', url: 'https://github.com/VijayPersistent/pipeline-project.git'
        }
      }
	  
	  stage ('Build')  {
	      steps {
          
            dir('java-source'){
            sh "mvn package"
          }
        }
         
      }
   
     stage ('SonarQube Analysis') {
        steps {
              withSonarQubeEnv('sonar') {
                
				dir('java-source'){
                 sh 'mvn -U clean install sonar:sonar'
                }
				
              }
            }
      }

    stage ('Artifactory configuration') {
            steps {
                rtServer (
                    id: "jfrog",
                    url: "http://3.235.230.13:8082/artifactory",
                    credentialsId: "jfrog"
                )

                rtMavenDeployer (
                    id: "MAVEN_DEPLOYER",
                    serverId: "jfrog",
                    releaseRepo: "iwayq-libs-release-local",
                    snapshotRepo: "iwayq-libs-snapshot-local"
                )

                rtMavenResolver (
                    id: "MAVEN_RESOLVER",
                    serverId: "jfrog",
                    releaseRepo: "iwayq-libs-release",
                    snapshotRepo: "iwayq-libs-snapshot"
                )
            }
    }

//     stage ('Deploy Artifacts') {
//             steps {
//                 rtMavenRun (
//                     tool: "maven-3.8.6", // Tool name from Jenkins configuration
//                     pom: 'java-source/pom.xml',
//                     goals: 'clean install',
//                     deployerId: "MAVEN_DEPLOYER",
//                     resolverId: "MAVEN_RESOLVER"
//                 )
//          }
//     }

//     stage ('Publish build info') {
//             steps {
//                 rtPublishBuildInfo (
//                     serverId: "jfrog"
//              )
//         }
//     }

    stage('Copy Dockerfile & Playbook to Ansible Server') {
            
            steps {
                  sshagent(['sshkey']) {
                       
                        sh "scp -o StrictHostKeyChecking=no Dockerfile ec2-user@44.204.234.10:/home/ec2-user"
                        sh "scp -o StrictHostKeyChecking=no create-container-image.yaml ec2-user@44.204.234.10:/home/ec2-user"
                    }
                }
            
        } 
    stage('Build Container Image') {
            
            steps {
                  sshagent(['sshkey']) {
                       
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@44.204.234.10 -C \"sudo ansible-playbook create-container-image.yaml\""
                        
                    }
                }
            
        } 
    stage('Copy Deployent & Service Defination to K8s Master') {
            
            steps {
                  sshagent(['sshkey']) {
                       
                        sh "scp -o StrictHostKeyChecking=no create-k8s-deployment.yaml ec2-user@34.238.248.178:/home/ec2-user"
                        sh "scp -o StrictHostKeyChecking=no nodePort.yaml ec2-user@34.238.248.178:/home/ec2-user"
                    }
                }
            
        } 

    stage('Waiting for Approvals') {
            
        steps{

				input('Test Completed ? Please provide  Approvals for Prod Release ?')
			  }
            
    }     
    stage('Deploy Artifacts to Production') {
            
            steps {
                  sshagent(['sshkey']) {
                       
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@34.238.248.178 -C \"sudo kubectl apply -f create-k8s-deployment.yaml\""
                        sh "ssh -o StrictHostKeyChecking=no ec2-user@34.238.248.178 -C \"sudo kubectl apply -f nodePort.yaml\""
                        
                    }
                }
            
        } 
         
   } 
}
