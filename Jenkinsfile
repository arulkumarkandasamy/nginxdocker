#!groovy

node () {
    timestamps {
      ansiColor('xterm') {
        step([$class: 'WsCleanup'])

        stage('Checkout') {
          String repo_url = 'https://github.com/arulkumarkandasamy/nginxdocker.git'

          checkout(
              changelog: true,
              poll: true,
              scm: [$class               : 'GitSCM',
                  branches             : [[name: 'master']],
                  doGenerateSubmoduleConfigurations: false,
                  userRemoteConfigs        : [[url: repo_url]]
              ]
          )

        }

      stage ('Build and Push Docker Image') {
        env.IMAGE_TAG = env.BUILD_NUMBER
        env.DOCKER_IMAGE = "arulkumar1967/kubernetes:" + env.BUILD_NUMBER
        sh '''
            env
            AWS_SECRET_ACCESS_KEY=eeq3pWMRLuggRVY8rQakdxjevGTey7sFHbKNon3U
            AWS_ACCESS_KEY_ID=AKIAIOLL6QZMBXNWC7FQ
            AWS_DEFAULT_REGION=eu-east-1
            export AWS_SECRET_ACCESS_KEY AWS_ACCESS_KEY_ID AWS_DEFAULT_REGION
            echo "Building container"
            sudo docker build -t ${IMAGE_TAG} .
            echo "Pushing container to registry"
            sudo docker login -u=arulkumar1967 -p=sumathi123
            sudo docker tag ${IMAGE_TAG} ${DOCKER_IMAGE}
            sudo docker push ${DOCKER_IMAGE}
        '''
      }

      stage ('Deploy') {
         withCredentials([[$class: "FileBinding", credentialsId: 'kubeconfig', variable: 'KUBE_CONFIG']]) {
            def kubectl = "kubectl  --kubeconfig=\$KUBE_CONFIG --context=useast1.k8s.vidsacsolutions.com"
            sh "${kubectl} run web-server --image=${DOCKER_IMAGE} --replicas=2 --port=80"
          }
      }
    }
  }
}