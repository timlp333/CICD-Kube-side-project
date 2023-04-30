pipeline {

    agent {
        label 'respberrypi'
    }
/*
	tools {
        maven "maven3"
    }
*/
    environment {
        registry = "timlp333/vproappdock"
        registryCredential = 'dockerhub'
    }

    stages{

        // stage('Fetch Code') {
        //     steps {
        //         git branch: 'paac', url: 'https://github.com/devopshydclub/vprofile-project.git'
        //     }
        // }
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        // stage('UNIT TEST'){
        //     steps {
        //         sh 'mvn test'
        //     }
        // }

        // stage('INTEGRATION TEST'){
        //     steps {
        //         sh 'mvn verify -DskipUnitTests'
        //     }
        // }

        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {

            environment {
                scannerHome = tool 'mysonarscanner4'
            }

            steps {
                withSonarQubeEnv('sonar-pro') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build App Image') {
            steps {
                script {
                    dockerImage = docker.build registry + ":V$BUILD_NUMBER"
                }
            }
        }

        stage('Upload Image') {
            steps {
                script {
                    dockerwithRegistry('', registryCredential) {
                        dockerImage.push("V$BUILD_NUMBER")
                        dockerImage.push('last')
                    }
                }
            }
        }

        stage('remove Unused docker Image') {
            steps {
                sh "docker rmi $registry:V$BUILD_NUMBER"
            }
        }
        
        stage('k8s deploy') {
            agent {label 'master'}
                steps {
                    sh "helm --kubeconfig /etc/rancher/k3s/k3s.yaml upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:V${BUILD_NUMBER} --n test"
                }
        }
    }


}
