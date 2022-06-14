pipeline {
  agent {
    kubernetes {
      namespace ''
      yaml """
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: docker
            image: artifactory-repository/docker:dind-daniel
            imagePullPolicy: Always
            tty: true
            securityContext:
              privileged: true
            volumeMounts:
              - name: docker-graph-storage
                mountPath: /var/lib/docker
          imagePullSecrets:
          - name: "artifactory"
          volumes:
          - name: workspace-volume
          - name: docker-graph-storage
            emptyDir: {}
        """.stripIndent()
    }
  }
  environment {
      CREDENTIALS = credentials('SA')
      VIRTUAL_REPO = 'VIRTUAL_REPO.com'
      
      VIRTUAL_REPO_NAME = 'test-virtual'
      scanConfig = ''
      server = ''
      buildInfo = ''
  }
 

  stages {
    stage('Build') {
        steps {
            container('docker') {
                script{
                  server = Artifactory.server 'MVP'
                  sh "echo 'this is our server'"
                  sh "echo ${server}"
                  def rtDocker = Artifactory.docker server: server

                  buildInfo = Artifactory.newBuildInfo()
                  buildInfo.project = 'de002'
                  sh 'docker login --username=${CREDENTIALS_USR} --password=${CREDENTIALS_PSW} ${VIRTUAL_REPO1}'
                  sh 'docker pull ${VIRTUAL_REPO1}/jenkins:latest'

                  sh "docker tag ${VIRTUAL_REPO1}/jenkins:latest ${VIRTUAL_REPO1}/jenkins:latest"
                  sh 'docker login --username=${CREDENTIALS_USR} --password=${CREDENTIALS_PSW} ${VIRTUAL_REPO}'
                  sh "docker push ${VIRTUAL_REPO1}/jenkins:latest"
                  sh "echo ${rtDocker}"
                  rtDocker.push 'test-virtual/test:latest', 'test-virtual', buildInfo

                  server.publishBuildInfo buildInfo
                  sh "echo ${buildInfo.name}"
                  sh "echo ${buildInfo.project}"
                  sh "echo ${buildInfo.number}"
                }
            }
        }
    }
    stage('XRAY') {
        steps {
            script {
                scanConfig = [
                'buildName' : buildInfo.name,
                'buildNumber' : buildInfo.number,
                'project': buildInfo.project
                    ]
                sh "echo 'this is our scan config'"
                sh "echo ${scanConfig}"
                def scanResult = server.xrayScan scanConfig
                echo scanResult as String
            }
        }
    }
   }
}
