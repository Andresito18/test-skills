
/// Pipeline variables
def CLUSTER_NAME="Falabella-Cluster"
def PROJECT_ID = "projectId"
def PROJECT_ID_QA = "qa"
def PROJECT_ID_PROD = "produccion" 
def imageTag = "gcr.io/${PROJECT_ID}/${JOB_NAME}:${BUILD_NUMBER}"
def JENKINS_MAIL = "jenkins@gmail.com"
def MAIL_CI = "mailci@gmail.com"
def MAIL_CD_DEV = "devexample@gmail.com"
def MAIL_CD_LAB = "devlabQA@gmail.com"
def MAIL_CD_PROD = "prodexample@gmail.com"

pipeline {
  options {
      timeout(time: 20, unit: 'MINUTES')
  }

/// 
  agent {
    kubernetes {
      label 'flask-slave'
      defaultContainer 'jnlp'
      yaml """
apiversion: v1
kind: Pod
metadata:
  labels:
    component: ci
spec:
  serviceAccountName: cd-jenkins
  volumes:
  - name: dockersock
    hostPath:
      path: "/var/run/docker.sock"
  - name: docker
    hostPath:
      path: "/usr/bin/docker"
  - name: google-cloud-key
    secret:
      secretName: registry-jenkins
  containers:
  - name: gcloud
    image: gcr.io/cloud-builders/gcloud
    volumeMounts:
    - name: google-cloud-key
      readOnly: true
      mountPath: "/var/secrets/google"
    - name: docker
      mountPath: "/usr/bin/docker"
    - name: dockersock
      mountPath: "/var/run/docker.sock"
    command:
    - cat
    env:
    - name: GOOGLE_APPLICATION_CREDENTIALS
      value: /var/secrets/google/key.json
    tty: true
  - name: python
    image: python
    command:
    - cat
    tty: true
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    volumeMounts:
    - name: google-cloud-key
      readOnly: true
      mountPath: "/var/secrets/google"
    command:
    - cat
    env:
    - name: GOOGLE_APPLICATION_CREDENTIALS
      value: /var/secrets/google/key.json
    tty: true
  - name: docker
    image: docker:17
    volumeMounts:
    - name: docker
      mountPath: "/usr/bin/docker"
    - name: dockersock
      mountPath: "/var/run/docker.sock"
    command:
    - cat
    tty: true
"""
    }
  }



  stages {

    // start login in environment dev

    stage('Initialize') {
        steps {
            container('docker') {
            sh 'docker --version'
            }
            container('gcloud') {
            sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
            sh "gcloud config set project ${PROJECT_ID}"
            }
        }
    }
    
    // download packages necessary for run test of app 

    stage('Build') {
      steps {
        container('python') {
            sh 'python -m pip install -r requirements.txt'
        }
      }
    }
    
    // Run Unit Test
    stage('Test') {
      steps {
        container('node') {
          sh 'python app/test.py'
        }
      }
    }

    // building image for deployment depend of branch that execute

    stage('Build-Image'){
      when {
        not { branch "master"}
      }
      steps {
        container('docker') {
          sh 'docker build --tag=${JOB_NAME}:${BUILD_NUMBER} .'
          sh 'docker images'
        }
      }
    }


    stage ('Build-Image Production') {
      when {
        branch "master"
      }
      steps {
        echo "${TAG_NAME}"
        container('docker') {
          sh 'docker build --tag=${JOB_NAME}:${TAG_NAME} .'
          sh 'docker images'
        }
      }
    }


    stage('Publish-Image'){
      when {
        not { branch "master"}
      }
      steps {
        container('docker') {
          sh "docker tag ${JOB_NAME}:${BUILD_NUMBER} gcr.io/${PROJECT_ID}/${JOB_NAME}:${BUILD_NUMBER}"
        }
        container('gcloud') {
          sh "gcloud docker -- push gcr.io/${PROJECT_ID}/${JOB_NAME}:${BUILD_NUMBER}"
        }
      }
    }

    stage ("Publish-Image Production") {
      when {
        branch "master"
        tag "v*"
      }
      steps {
        container('docker') {
          sh "docker tag ${JOB_NAME}:${TAG_NAME} gcr.io/${PROJECT_ID}/${JOB_NAME}:${TAG_NAME}"
        }
        container('gcloud') {
          sh "gcloud docker -- push gcr.io/${PROJECT_ID}/${JOB_NAME}:${TAG_NAME}"
        }
      }
    }



    stage('Deploy develop') {
      // Developer Branches
      when { branch 'develop' }
      steps {
        container('kubectl') {
          sh "gcloud container clusters get-credentials ${CLUSTER_NAME} --zone us-central1-a --project ${PROJECT_ID}"
          sh "sed -i.bak 's#${imageTag}#' ./k8s/deployment.yaml"
          sh 'kubectl apply -f ./k8s/deployment.yaml'
        }
        script {
          try {
            mail from: "${JENKINS_MAIL}",
                 to: "${MAIL_CD_DEV}",
                 subject: "Despliegue realizado en Ambiente Desarrollo ${JOB_NAME}-${BUILD_DISPLAY_NAME}",
                 body: "Se ha realizado despliegue automatico en el ambiente de desarrollo."
          }
          catch (exec) {
            echo 'Problema al enviar el correo'
          }
        }
      }
    }
     
    stage ('Aprobacion QA') {
      when { branch 'qa'}
      steps {
        script {
          try {
            mail from: "${JENKINS_MAIL}",
                 to: "${MAIL_CD_LAB}",
                 subject: "Aprobacion Requerida ${JOB_NAME}-${BUILD_DISPLAY_NAME}",
                 body: "Se ha realizado la construccion de ${JOB_NAME}, se requiere su aprobacion para desplegar en ambiente de QA, para aprobar ${BUILD_URL}"
          }
          catch (exec) {
            echo 'Problema al enviar el correo'
          }
        }
          timeout(time:5, unit:'DAYS'){
            input message: 'Aprueba Despliegue Ambiente QA?',
            submitter: 'DevOps'
          }
      }
    }

    stage('Deploy QA') {
      when { branch 'qa'}
      steps {
        container ('kubectl') {
          sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
          sh "gcloud config set project ${PROJECT_ID_QA}"
          sh "gcloud container clusters get-credentials ${CLUSTER_NAME} --zone us-central1-a --project ${PROJECT_ID_QA}"
          sh "sed -i.bak 's#${imageTag}#' ./k8s/deployment.yaml"
          sh 'kubectl apply -f ./k8s/deployment.yaml'
        }
      }
    }


    stage ('Aprobacion Produccion') {
      when {
        branch "master"
        tag "v*"
      }
      steps {
        mail from: "${JENKINS_MAIL}",
             to: "${MAIL_CD_PROD}",
             subject: "Aprobacion Requerida ${JOB_NAME}-${BUILD_DISPLAY_NAME}",
             body: "Se ha realizado la construccion de ${JOB_NAME}, se requiere su aprobacion para desplegar en ambiente de Produccion, para aprobar ${BUILD_URL}"
          timeout(time:5, unit:'DAYS'){
            input message: 'Aprueba Despliegue Ambiente Produccion?',
            submitter: 'DevOps'
          }
      }
    }


    stage ('Despliegue Produccion') {
      when {
        branch "master"
        tag "v*"
      }
      steps {
        container ('kubectl') {
          sh 'gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS'
          sh "gcloud config set project ${PROJECT_ID_PROD}"
          sh "gcloud container clusters get-credentials ${CLUSTER_NAME} --zone us-east1-d --project ${PROJECT_ID_PROD}"
          sh "sed -i.bak 's#gcr.io/${PROJECT_ID}/${JOB_NAME}:${TAG_NAME}#' ./k8s/deployment.yaml"
          sh 'kubectl apply -f ./k8s/deployment.yaml'
        }
      }
    }


  }



  post {
    always {
      echo "Pipeline Finalizado"
    }
    aborted {
      echo "Pipeline Abortado"
      SendEmail("El Pipeline fue abortado, por favor revisar la informacion en ${BUILD_URL}console")
    }
    failure {
      echo "Pipeline Fallido"
      SendEmail("Ocurrio un fallo en la ejecucion de Pipeline ${JOB_NAME}-${BUILD_DISPLAY_NAME}, por favor revisar la informacion en ${BUILD_URL}")
    }
    success {
      echo "Pipeline Exitoso!!"
      SendEmail("El Pipeline ${JOB_NAME}-${BUILD_DISPLAY_NAME} ha finalizado exitosamente. para mas informacion ver ${BUILD_URL}")
    }
  }

}
