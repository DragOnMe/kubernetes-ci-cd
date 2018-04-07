node {

    checkout scm

    env.DOCKER_API_VERSION="1.23"
    
    sh "git rev-parse --short HEAD > commit-id"

    tag = readFile('commit-id').replace("\n", "").replace("\r", "")
    appName = "hello-kenzan"
    # registryHost = "127.0.0.1:30400/"
    registryHost = "docker.io/drlee001/"
    imageName = "${registryHost}${appName}:${tag}"
    podNamespace = "default"
    env.BUILDIMG=imageName

    stage "Build"
    
        sh "docker build -t ${imageName} -f applications/hello-kenzan/Dockerfile applications/hello-kenzan"
    
    stage "Push"

        sh "docker push ${imageName}"

    stage "Deploy"

        #sh "sed 's#127.0.0.1:30400/hello-kenzan:latest#'$BUILDIMG'#' applications/hello-kenzan/k8s/deployment.yaml | kubectl apply -f -"
        sh "sed 's#127.0.0.1:30400/hello-kenzan:latest#'$BUILDIMG'#' applications/hello-kenzan/k8s/deployment.yaml | kubectl apply -f -"
        sh "kubectl set image -n ${podNamespace} deployment/${K8S_DEPLOYMENT_NAME} ${K8S_DEPLOYMENT_NAME}=${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
        sh "kubectl rollout status deployment/hello-kenzan"
}

#!groovy
podTemplate(label: 'hello-kenzan', containers: [
    containerTemplate(name: 'kubectl', image: 'smesch/kubectl', ttyEnabled: true, command: 'cat',
        volumes: [secretVolume(secretName: 'kube-config', mountPath: '/root/.kube')]),
    containerTemplate(name: 'docker', image: 'docker', ttyEnabled: true, command: 'cat',
        envVars: [containerEnvVar(key: 'DOCKER_CONFIG', value: '/tmp/'),])],
        volumes: [secretVolume(secretName: 'docker-config', mountPath: '/tmp'),
                  hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')
  ]) {

    node('hello-kenzan') {

        def DOCKER_HUB_ACCOUNT = 'drlee001'
        def DOCKER_IMAGE_NAME = 'hello-kenzan'
        def K8S_DEPLOYMENT_NAME = 'hello-kenzan'
        def POD_NAMESPACE = 'ns-jenkins'

        stage('Clone hello-kenzan App Repository') {
            checkout scm
            
            container('docker') {
                stage('Docker Build & Push Current & Latest Versions') {
                    sh ("docker build -t ${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} .")
                    sh ("docker push ${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}")
                    sh ("docker tag ${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER} ${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:latest")
                    sh ("docker push ${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:latest")
                }
            }

            container('kubectl') {
                stage('Deploy New Build To Kubernetes') {
                    sh ("kubectl set image -n ${POD_NAMESPACE} deployment/${K8S_DEPLOYMENT_NAME} ${K8S_DEPLOYMENT_NAME}=${DOCKER_HUB_ACCOUNT}/${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}")
                }
            }

        }        
    }
}
