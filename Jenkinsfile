pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: docker
            image: docker:latest
            command:
            - cat
            tty: true
            volumeMounts:
             - mountPath: /var/run/docker.sock
               name: docker-sock
          volumes:
          - name: docker-sock
            hostPath:
              path: /var/run/docker.sock    
        '''
    }
  }
  environment {
        SECRET_KEY     = credentials('secret-key')
        GIT_CRED       = credentials('git-creds')
  }
  stages {
    stage('Clone') {
      steps {
        container('docker') { 
          
    
          checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'docfile']], userRemoteConfigs: [[url: 'https://github.com/harshgoenka-cloudside/jenkins-build-dockerfile.git']]])
          dir ('docfile') {
          sh 'cat $SECRET_KEY | docker login -u _json_key --password-stdin https://us-central1-docker.pkg.dev'
          sh 'docker build -t us-central1-docker.pkg.dev/cloudside-academy/harsh-test-repo/jenkins-argo:v$BUILD_NUMBER .'
          sh 'docker push us-central1-docker.pkg.dev/cloudside-academy/harsh-test-repo/jenkins-argo:v$BUILD_NUMBER'
          }
        
         sh 'apk add git curl bash'
         sh 'curl -s "https://raw.githubusercontent.com/kubernetes-sigs/kustomize/master/hack/install_kustomize.sh"  | bash'
         sh 'mv kustomize /usr/local/bin'
         
          checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'kusfile']], userRemoteConfigs: [[url: 'https://github.com/harshgoenka-cloudside/argocd-kustomize.git']]])
          
          sh 'chown -R root kusfile'
          
          dir ('kusfile'){
            sh 'kustomize edit set image us-central1-docker.pkg.dev/cloudside-academy/harsh-test-repo/jenkins-argo=*:v$BUILD_NUMBER && kustomize build .'
            sh "git config user.email harsh.goenka@thecloudside.com"
            sh "git config user.name harshgoenka-cloudside"
            sh "git checkout -b main"
            sh 'git add kustomization.yaml'
            sh "git commit -m 'updated kustomize file'"
            sh 'git status'
            sh 'git branch -a'
            sh 'git push https://$GIT_CRED_USR:$GIT_CRED_PSW@github.com/harshgoenka-cloudside/argocd-kustomize.git'
          }
          
        }
      }
    }  
  }
}
