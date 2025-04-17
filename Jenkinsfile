pipeline {
     agent
    {
        label 'Linux_Slave-new'
    }

    environment {
        DOCKER_REGISTRY = 'https://hub.docker.com'
        JAVA_HOME = '/usr/lib/jvm/openlogic-openjdk-21-hotspot'
        MAVEN_HOME = '/data/admin/jenkins/.jenkins/tools/hudson.tasks.Maven_MavenInstallation/Maven_3.9.2'
        // KUBE_CONFIG = credentials('kubeconfig')
        // DOCKER_USER = thavamani0212
        // DOCKER_PASS = Thavamani126@
        DOCKER_SERVICE = 'Docker'
        SERVICE_NAME = 'productcatalogue'
        SERVICE_PORT = '8020'
        SONAR = 'Sonar'
        SCANNER_HOME = tool 'SonarQube Scanner 4.7';

    }
     triggers {  
        GenericTrigger(
            genericVariables: [],
            token: 'mani@12three', 
            printContributedVariables: true,
            printPostContent: true
        )
    }

    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build and Compile the Code') {
            steps {
                script {
                    sh "${MAVEN_HOME}/bin/mvn clean package"
               }

            }
        }
        stage('SonarQube Analysis') {
            steps
            {
                withSonarQubeEnv("${SONAR}")
                {
                    sh """

                        ${MAVEN_HOME}/bin/mvn sonar:sonar \

                        -Dsonar.projectKey=productcatalogue \

                        -Dsonar.projectName=Product Catalogue \
                        
                        -Dsonar.login=sqa_50fd1180a9b3bab8a6763dfd338c6099e466af21

                    """

                }
            }
        }
        stage('Sonar Quality Gate checks'){
            steps {
                script {
                  waitForQualityGate abortPipeline: true, credentialsId: 'sonar-new-2024'
                }
            }
        }
      
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_REGISTRY}/${SERVICE_NAME}:${BUILD_NUMBER} ."
            }
        }
        
        stage('Scan Docker Image') {
            steps {
                sh '''
                    echo "Scanning image for vulnerabilities..."
                    trivy image --severity CRITICAL,HIGH ${DOCKER_REGISTRY}/${SERVICE_NAME}:${BUILD_NUMBER}
                '''
            }
        }
        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-registry-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "docker login -u $DOCKER_USER -p $DOCKER_PASS ${DOCKER_REGISTRY}"
                    sh "docker push ${DOCKER_REGISTRY}/${SERVICE_NAME}:${BUILD_NUMBER}"
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    dir('../kubernetes') {
                        sh "kubectl apply -f ${SERVICE_NAME}-service.yaml"
                        sh "kubectl set image deployment/${SERVICE_NAME} ${SERVICE_NAME}=${DOCKER_REGISTRY}/${SERVICE_NAME}:${BUILD_NUMBER}"
                    }
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh "kubectl rollout status deployment/${SERVICE_NAME}"
                    sh "curl -s http://localhost:${SERVICE_PORT}/health"
                }
            }
        }
    }
    
    post {
        success {
            mail to: 'n.thavamani@nagarro.com',
                subject: "SUCCESS | Deployment Completed for ${JOB_BASE_NAME}",
                body: "Hi,\n\n\nThe deployment has been completed for the Jenkins pipeline '${JOB_BASE_NAME}'.\n\n\nThe Application URL and Pipeline details are below.\n\n\nStatus: SUCCESS\n\nPipeline: ${JOB_NAME}\n\nBuild No: ${BUILD_NUMBER}\n\n"
        }
        failure {
            mail to: 'n.thavamani@nagarro.com',
                subject: "Failed | Deployment  Failed for ${JOB_BASE_NAME}",
                body: "Hi,\n\n\nThe deployment has failed for the Jenkins pipeline '${JOB_BASE_NAME}'. Please check, Below are the details.\n\n\nStatus: FAILED\n\nPipeline: ${JOB_NAME}\n\nBuild No: ${BUILD_NUMBER}\n\nBuild URL: ${BUILD_URL}\n\n\n"
        }
    }
}
