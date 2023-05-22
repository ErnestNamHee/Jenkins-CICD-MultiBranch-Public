pipeline  {
    agent any

    environment {
        FE_SWR_CREDENTIALS_LOGIN    = credentials('fe-swr-credential-login')
        FE_SWR_CREDENTIALS_PASSWORD = credentials('fe-swr-credential-password')
        FE_SWR_URL = "registry.eu-west-0.prod-cloud-ocb.orange-business.com"
        FE_SWR_ORGANIZATION = "ernest"
        NGINX_DOCKER_IMAGENAME = "nginx-custom-image"
        
        
        
    }

    stages {
        stage ('Build') {
            steps {
                echo "Building the NGINX Custom Docker Image"
                //def customImage = docker.build("my-nginx:${env.BUILD_ID}")        
                sh "docker build -t ${FE_SWR_URL}/${FE_SWR_ORGANIZATION}/${NGINX_DOCKER_IMAGENAME}:${env.BUILD_ID} ./Build/nginx/."
            }
        }
        stage ('Register') {
            steps {
                echo "Register the NGINX Custom Docker Image to SWR"              
                sh "docker login -u ${FE_SWR_CREDENTIALS_LOGIN} -p ${FE_SWR_CREDENTIALS_PASSWORD} ${FE_SWR_URL}"
                
                sh "docker push ${FE_SWR_URL}/${FE_SWR_ORGANIZATION}/${NGINX_DOCKER_IMAGENAME}:${env.BUILD_ID}"
                sh "docker tag ${FE_SWR_URL}/${FE_SWR_ORGANIZATION}/${NGINX_DOCKER_IMAGENAME}:${env.BUILD_ID} ${FE_SWR_URL}/${FE_SWR_ORGANIZATION}/${NGINX_DOCKER_IMAGENAME}:latest"
                sh "docker push ${FE_SWR_URL}/${FE_SWR_ORGANIZATION}/${NGINX_DOCKER_IMAGENAME}:latest"
                
            }
        }
        stage ('Deploy to FE CCE') {
            steps {
                echo "Deploy to CCE"
                withKubeConfig([credentialsId: 'CCEKubeconfigDemo4Secret']) {
                    sh "kubectl delete deployment.apps/nginx-apps"
                    sh "kubectl create  -f ./Deployments/nginx/AppsDeployment.yaml"
                    //sh "kubectl delete service/nginx"
                    sh "kubectl create  -f ./Deployments/nginx/AppsService.yaml"
                    
                }
            }
        } 
    }
}
