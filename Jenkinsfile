node {
  stage('Checkout') {
    checkout scm
  }

  //stage('Verify') {
  //  withCredentials([
  //      file(credentialsId: 'keybase-envfile',
  //           usernameVariable: 'KEYBASE_ENV_FILE')]) {
  //    def scmUrl = scm.getUserRemoteConfigs()[0].getUrl()
  //    sh '''
  //      docker run -it --env-file=${KEYBASE_ENV_FILE} \
  //        -e KEYBASE_TRUSTED_USERS=lukebond \
  //        -e GIT_USER_EMAIL="luke.n.bond+bot@gmail.com" \
  //        -e GIT_REPO="${scmUrl}" \
  //        -e GIT_REVISIONS_TO_VERIFY=1 \
  //        controlplane/keybase:latest
  //    '''
  //  }
  //}

  stage('Build') {
    withCredentials([
        usernamePassword(credentialsId: 'docker-credentials',
                         usernameVariable: 'USERNAME',
                         passwordVariable: 'PASSWORD')]) {
      sh 'docker image build -t ${USERNAME}/demo-api:latest .'
    }
  }

  stage('Push') {
    withCredentials([
        usernamePassword(credentialsId: 'docker-credentials',
                         usernameVariable: 'USERNAME',
                         passwordVariable: 'PASSWORD')]) {
      sh 'docker login -p "${PASSWORD}" -u "${USERNAME}"'
      sh 'docker image push ${USERNAME}/demo-api:latest'
    }
  }

  stage('Kubesec') {
    sh """
      echo 'Running Kubesec...'

      kubesec () {
        echo function args: "${1}"
        local FILE="${1}";
        [[ ! -f "${FILE}" ]] && {
            echo "kubesec: ${FILE}: No such file" >&2;
            return 1
        };
        curl --silent \
          --compressed \
          --connect-timeout 5 \
          -F file=@"${FILE}" \
          https://kubesec.io/
      }
      
      if kubesec ./deployment.yaml | jq --exit-status '.score > 10' >/dev/null; then
        exit 0;
      fi

      echo 'The application failed on kubesec score'
      exit 1
    """
  }

  stage('Deploy') {
    withCredentials([
        file(credentialsId: 'kube-config',
             variable: 'KUBECONFIG')]) {
      sh 'kubectl apply -f deployment.yaml'
    }
  }
}
