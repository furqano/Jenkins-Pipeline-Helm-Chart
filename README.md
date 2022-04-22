# Jenkins-Pipeline-Helm-Chart

* An Jenkins Pipeline to build docker image for RestApi-app , Push it to Docker Regestry and relaunch helm chart (restapi-app) to see the updated image changes in an end-to-end deployment.

## Pre-Reqs
* Mongodb Helm chart 
* Mongodb URL - Need to be passed using parametes set for restapi-app
* Dockerfile  - To build the image
* Helm Chart  - [Serversapi-app](https://github.com/furqano/Helm-Chart-Server2Api)

## Slave Nodes
## Created 2 slave nodes to manage the pipeline dependencies conflicts.

![image](https://user-images.githubusercontent.com/64476159/164753063-c6435e05-2f3a-42df-8e17-bbb953c0abbd.png)

* Ec2 running with Docker and Maven - This Node is used to build the .jar file for restapi-app once the repository is cloned to workspace. Using Dockerfile creating an image with the updated .jar file and pushing it to Docker Registry.

* Windows running with helm and k8s - This is used to relaunch the helm chart to get the updated Image from docker registry and deploy the restapi-app.

## Plugins Used
* Docker Pipeline
* Maven integration
* Windows agents

Created Credentials for DockerHub to push the images to repositories.
![image](https://user-images.githubusercontent.com/64476159/164772684-1ce144b9-3c14-4953-9ce5-14258cb1e3b0.png)


## Pipeline

```
pipeline{
    agent any
    environment { 
        registry = "furqano/restapi-app" 
        registryCredential = 'dockhub' 
        dockerImage = '' 
    }
    stages{
      stage('Clone Repo') {
      agent { label 'docker' }
        steps {
            git branch: '**', url: 'https://github.com/furqano/Java-Rest-API-2-Server.git'
            
            }
        }
      stage('Build app') {
      agent { label 'docker' }
        steps {
            withMaven(maven :'maven'){
                sh "mvn package"
            }
            
            }
        }
      stage('Build Image'){
            agent { label 'docker' }
            steps{
                
            
            script{
                dockerImage = docker.build("furqano/test")
            }
        }}
        stage('Deploy our image') { 
            agent { label 'docker' }
            steps { 
                script { 
                    docker.withRegistry('https://registry.hub.docker.com', 'dockhub') {            
       dockerImage.push("latest")        
              }    
                   
                    }
                } 
            }
            stage("Helm deploy"){
                agent{ label 'helm'}
                steps{
                    
                
                script{
                    bat "helm upgrade dep2 -n dep test  "
                }}
            }
        } 
    }
```

## Screenshots
![image](https://user-images.githubusercontent.com/64476159/164774326-b69b2df3-4ba3-426a-925f-70e479fb738a.png)

## Maven Build
![image](https://user-images.githubusercontent.com/64476159/164750838-8da30969-b523-42bc-98b3-2199ff4eff58.png)

## Before Edit - Previous Restapi-app Image

![finding resources by name](https://user-images.githubusercontent.com/64476159/164773110-69f744b1-892d-4020-81d4-70594f6ef255.png)

## After Edit - Running Jenkins Job to deploy the new Image 

![Jenkins helm sucess](https://user-images.githubusercontent.com/64476159/164773195-07388801-08ac-4f09-9c2d-9082fdb85ebc.png)

## New Updated Image deployed

![Modified to get value by name ](https://user-images.githubusercontent.com/64476159/164773228-55dbc4a0-6349-4433-abe0-9a9b58f15b62.png)


