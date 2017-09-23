podTemplate(label: 'mypod', containers: [
    containerTemplate(name: 'docker', image: 'docker:latest', ttyEnabled: true, command: 'cat')],
    volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]) {

  node('mypod') {
      stage('Build and Push Webserver'){
          container('docker'){
           git url: 'git://github.com/pblaas/docker_workshop_container.git'
           dir('src'){
             git url: 'git://github.com/pblaas/workshop_fe.git'
           }
            stage ('Build Docker image'){
              def imageName = "${env.DOCKERHUB_USER}/${env.NGINXCONTAINER_IMAGE}:${env.BUILD_TAG}"
              sh "docker build -t ${imageName}  ."
              def img= docker.image(imageName)

              stage ('Push Docker image to registry'){
                docker.withRegistry("https://registry.hub.docker.com", "docker-registry") {
                  img.push()
                }
              }
            }
          }
      }
      stage('Build and Push colorservice'){
          container('docker'){
           dir('src'){
             git url: 'git://github.com/pblaas/workshop_colorservice.git'
           }
           git url: 'https://github.com/pblaas/docker_workshop_container.git'
            stage ('Build Docker image'){
              def imageName = "${env.DOCKERHUB_USER}/${env.COLORSERVICE_IMAGE}:${env.BUILD_TAG}"
              sh "docker build -t ${imageName}  ."
              def img= docker.image(imageName)

              stage ('Push Docker image to registry'){
                docker.withRegistry("https://registry.hub.docker.com", "docker-registry") {
                  img.push()
                }
              }
            }
          }
      }
    }
}

