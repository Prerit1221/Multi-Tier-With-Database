pipeline {
    agent any

    tools {
        maven 'maven3'
        jdk 'jdk17'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('Git Checkout') {
            steps {
                echo 'Code Checkout'
                git branch: 'start', url: 'https://github.com/Prerit1221/Multi-Tier-With-Database.git'
            }
        }

        stage('Debug Environment') {
            steps {
                sh '''
                echo "JAVA_HOME=$JAVA_HOME"
                which java
                which javac
                java -version
                javac -version
                mvn -version
                '''
            }
        }

        stage('Compile') {
            steps {
                echo 'Compiling the code'
                sh 'mvn compile'
            }
        }

        stage('Test Cases') {
            steps {
                echo 'Running Test Cases'
                sh 'mvn test -DskipTests=true'
            }
        }

        stage('Trivy File System Scan') {
            steps {
                echo 'Running Trivy File System Scan'
                sh 'trivy fs --format table -o trivy-report.txt .'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube Analysis'

                withSonarQubeEnv('sonarscanner') {
                    sh """
                    $SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectName=MultiTier \
                    -Dsonar.projectKey=FullStack-Blogging-App \
                    -Dsonar.sources=. \
                    -Dsonar.java.binaries=target/
                    """
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Building the code'
                sh 'mvn package -DskipTests=true'
            }
        }

        stage('Publish to Nexus') {
            steps {
                echo 'Publishing to Nexus Repository'

                withMaven(
                    globalMavenSettingsConfig: 'settings-maven',
                    jdk: 'jdk17',
                    maven: 'maven3',
                    traceability: true
                ) {
                    sh 'mvn deploy -DskipTests=true'
                }
            }
        }

        stage('Docker Build Image') {
            steps {
                echo 'Building Docker Image'

                withDockerRegistry(
                    credentialsId: 'Dockercred',
                    url: 'https://index.docker.io/v1/'
                ) {
                    sh 'docker build -t preritsharma/fullstack-blogging-app:latest .'
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                echo 'Running Trivy Docker Image Scan'
                sh 'trivy image --format table -o trivyimage-report.txt preritsharma/fullstack-blogging-app:latest'
            }
        }

        stage('Docker Push') {
            steps {
                echo 'Pushing Docker Image to Docker Hub'

                withDockerRegistry(
                    credentialsId: 'Dockercred',
                    url: 'https://index.docker.io/v1/'
                ) {
                    sh 'docker push preritsharma/fullstack-blogging-app:latest'
                }
            }
        }

        stage('Deployment to K8') {
            steps {
                echo 'Running Deployment to Kubernetes'

                kubeconfig(
                    credentialsId: 'k8token',
                    serverUrl: 'https://C46842466A7BA49528B7ED6EE8F13DA3.gr7.ap-south-1.eks.amazonaws.com',
                    caCertificate: '''-----BEGIN CERTIFICATE-----
MIIDBTCCAe2gAwIBAgIIBdICAUlLI14wDQYJKoZIhvcNAQELBQAwFTETMBEGA1UE
AxMKa3ViZXJuZXRlczAeFw0yNjA2MDgwNzQxNTRaFw0zNjA2MDUwNzQ2NTRaMBUx
EzARBgNVBAMTCmt1YmVybmV0ZXMwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEK
AoIBAQDNvbwI6Dl87+q4XhlfgyvA6/cfuERqB0a3eZ/oTAhX5kdevICHu9wWzih/
OLFN9u/CCkf7yi3B3j/Eg2MqD83z4l5FIVVoizVUWnr8YFTTsmJEtLcGFPAr9Eb0
KSOq6noezEVHLmhoD8sN7ZEGY71ThVPlxGl0eCT2Rk6nMP0OXv2isAA1dYfB+HFg
i6inT8u7/sUh2xBHqy/LKS3Cqti6PXSoRAL1OcKgUYZX4A1aaGtjT1qsD+WDAJ+S
ipJ1Rl5HFq8PCPftmqVG4LSfazpoAxbdiCzKuJl3IwxyJ51xWNfmurGlyIYCrR5d
Y0Y7PNU2xyr8B9PEJAnoxy9kfdiXAgMBAAGjWTBXMA4GA1UdDwEB/wQEAwICpDAP
BgNVHRMBAf8EBTADAQH/MB0GA1UdDgQWBBSudbSf168/5FR1TUEzanL+BtT7YDAV
BgNVHREEDjAMggprdWJlcm5ldGVzMA0GCSqGSIb3DQEBCwUAA4IBAQCeXBnf0EE2
JrbmGeO1IIwByTUm164NUXGerf14nb0g0xBhL5dMHtkNAWW9e7jWoL8Jagsd2bAf
tji3s53CG9osV5aexRec2B+X9adr8kauDHH6fpZPINuPPtl35i5ZeM7yrCLLPsyg
v+pbKRb+TPIMMuKahV3tP1DLSexm0lhGTJUPd9+TEI16U0/MjfnaF6RnMKjZPjx+
Px928xURvW3ECjfEq73B4YhL2ctxMfjK+arKOKDGdvuwF/3kJgHdGSY9iF2YgXsb
WAVlYmr871Bdc6Y58TLroFJUhtkptif4tBhyttyM9BITqbWZy2InyrooTVBRgOYw
RfCi9SrHlziZ
-----END CERTIFICATE-----'''
                ) {
                    sh 'kubectl apply -f ds.yml -n webapps'
                    sleep 30
                }
            }
        }

        
    }
}
