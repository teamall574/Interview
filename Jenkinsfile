pipeline {
    agent {
        lable "Anji"  ##need to give slave name slave name is Anji
    }
    environment {
        registryCredential = "anjitest"
        dockerimagename = "anji1592/kubetest"
        dockerImage = " "
    } //or use this
    environment {
        app_name = "zomato-app"
        release = "1.0.0"
        image_name = "${anji1592}" + "/" + "${app_name}"
        iamge_tag = "${release}-${BUILD_NUMBER}" 

    }
    stages {
        stage("clean workspace"){
            steps {
                cleanWS()  ##it will clear the workspace every build
            }
        }
    
        stages {
            stage('checkout source') {
                steps {
                    git 'https://github.com/teamall574/test.git'   ##it is public repository
                }
            }
            stage (clone) {
                steps{     ##if it is private repository if not then not need credentials
                    git credentialsId: 'git_credentials', url: 'https://github.com/teamall574/test.git', branch:'dev'  #if you want add another barnch corner add 
                }
            } 
        }
        stage('sonar quality status'){
            script{
                withsonarQubeEnv(credentialsId: 'sonar-token') {
                sh 'mvn sonar:sonar' #it will send the code into the sonarqube
                sh 'npm run sonar' #It will send the code into the sonarqube you need to set the sonar:sonarqube-scanner in package.json andd version also
                }
            }
        }
        stage('build angular application) {
            steps {
                sh 'nvm install' 
                sh 'node  version'
                sh 'npm install -g @angular/cli'
                sh 'ng v'
                sh 'ng build'   ##this will use to build the angular application              
            }  
        }
        stage('quality gate staus') {
            steps {
                script{
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage (maven build##need to install maven and configure in jenkins system) {
            steps{
                sh 'mvn clean package'
            }
        }
        stage (maven build##need to install maven and configure in jenkins system) {
            steps{
                sh 'mvn test'
            }
        }

        stage('OWASP-CHECK'){  ##it will check owasp top 10 issues in jar file or package application
            steps {
                sh "dependencyCheck additionalArgumnets: '--scan target/', odcInstallation: 'owasp' dependencyCheckPublisher pattern: '**/dependency-check-report.xml'"
            }
        }
        stage {
            steps{ ##this using pom.xml configuartion in add ip add or else you can use the pipeline syntax to create the syntax and add username and password also 
            sh 'mvn deploy' ##this will copy your artifacts into nexus repository
            }
        }
        stage (tomcat server##need ssh key to connect the tomcat server and the code push tomcat server) {
            steps{
                scp webapp/target/webapp.war ec2-user@25..15.25.24:/opt/tomcat/webapps
            }
        }
        stage('build image') {
            steps {
                script {
                    dockerImage = docker.build dockerimagename
                }
            }
        }
        stage('push image') {
            steps {
                script {
                    docker.withRegistry( 'https://registry.hub.docker.com', registryCredential) {
                        dockerImage.push("latest")
                    }
                }
            }
        }
        stage('get cedentials'){  ##need to add first vault plugin url and then add username and password of vault
            withVault(configuration: [timeout: 60, vaultCredentialId: 'vaultId', vaultUrl: 'https://172.65.45.3.anji.com'], valueSecret: [path: 'secret/dockercred'], secretValues: [valuekey: 'username'], [valuekey: 'password']) 
            sh 'echo $username' 
        }   
        stage('Docker Build & Push') {  ##this second method 
            steps {
                script{
                    withDockerRegistry(credentialsId: '2fe19d8a-3d12-4b82-ba20-9d22e6bf1672', toolName: 'docker') {
                        
                        sh "docker build -t anji ."
                        sh "docker tag  anji anji1592/kubetest:latest"
                        sh "docker push anji1592/kubetest:latest"
                    }
                }
            }
        }
        stage ('docker build'){ 
            steps {
                docker_image = docker.build "${image_name}"
            }
        }
        stage ('push docker image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '2fe19d8a-3d12-4b82-ba20-9d22e6bf1672', toolName: 'docker') {
                        docker_image.push("${image_tag}")
                        docker_image.push('latest')
                }
            }
        }
        stage('test docker image') {
            steps {
                sh "trivy --version"  
                sh "trivy image anji1592/kubetes"
                sh "bash anji.sh"
            }
        }
        stage('k8s deploy') {
        steps{
            withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
		sh "sed -i '/^\\s*image:/ s|.*|  image: ${image_name}|' ${deployment.yml}"  //it will replace the docker image tag or build number everytime in deployment file
                sh 'kubectl get pods'
                sh 'kubectl create -f deployment.yml'
            }
          }
        }
        post {
            success {
                slackSend "Build deployed successfully - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            }
        }
        post {
            failure {
              slackSend failOnError:true message:"Build failed  - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
            }
        }
        //or else you can send the mail also
        post {
            
        }
    }
}
