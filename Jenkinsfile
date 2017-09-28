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
node{
  stage('Deploy to cluster'){
    sh("wget -q https://storage.googleapis.com/kubernetes-release/release/v1.6.1/bin/linux/amd64/kubectl && chmod +x kubectl")
    sh("./kubectl config set-credentials jenkins-build --token=`cat /var/run/secrets/kubernetes.io/serviceaccount/token`")
    sh("./kubectl config set-cluster internal1 --server=https://10.3.0.1 --certificate-authority=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt")
    sh("./kubectl config set-context default --user=jenkins-build --namespace=`cat /var/run/secrets/kubernetes.io/serviceaccount/namespace`  --cluster=internal1")
    sh("./kubectl config use-context default")

    sh("./kubectl -n default set image deployment workshopnginx workshopnginx=${env.DOCKERHUB_USER}/${env.NGINXCONTAINER_IMAGE}:${env.BUILD_TAG}")
    sh("./kubectl -n default set image deployment workshopcolorservice workshopcolorservice=${env.DOCKERHUB_USER}/${env.COLORSERVICE_IMAGE}:${env.BUILD_TAG}")
  }
}


