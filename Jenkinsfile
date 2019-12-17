def label = "mypod-${UUID.randomUUID().toString()}"
def serviceaccount = "jenkins-admin"
podTemplate(label: label, serviceAccount: serviceaccount,
containers: [containerTemplate(name: 'kubectl', image: 'smesch/kubectl', ttyEnabled: true, command: 'cat',
                             volumes: [secretVolume(secretName: 'kube-config', mountPath: '/root/.kube')]),
    containerTemplate(name: 'docker', image: 'docker:19.03', ttyEnabled: true, command: 'cat')],
                             volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]
) {

 

    node(label) {
        def GIT_URL= 'http://gitlab.ethan.svc.cluster.local:8084/gitlab/$TRAINING_USER/django-pipeline.git'
        def GIT_CREDENTIAL_ID ='gitlab-$TRAINING_USER'
        def GIT_BRANCH='master'

 

        stage('Git Checkout'){
            git branch: GIT_BRANCH, url: GIT_URL, credentialsId: GIT_CREDENTIAL_ID
        }

 

        stage('Docker Build') {
            container('docker'){
               sh("docker build -t localhost:32121/${TRAINING_USER}/django-pipeline/django:${env.BUILD_NUMBER} --network=host .")
            }
        }

 

        stage('Push Container Image') {
            container('docker'){
                withDockerRegistry([ credentialsId: "gitlab-user", url: "http://localhost:32121" ]) {
                   sh ("docker push localhost:32121/${TRAINING_USER}/django-pipeline/django:${env.BUILD_NUMBER}")
                   sh ("echo ${env.BUILD_NUMBER}")
                }
            }
        }
        
        stage('Deploy build to Kubernetes') {
            container('kubectl') {
                try{
                    sh ("kubectl create ns $TRAINING_USER")
                    sh ("kubectl get secret gitcred --namespace=ethan --export -o yaml | kubectl apply --namespace=$TRAINING_USER -f -")
                    //sh ("sed -i 's/${image}/localhost:32121/puneeth/django-pipeline/django:${env.BUILD_NUMBER}/g' deployment.yaml")
                    sh ("kubectl get deployment/django -n ${TRAINING_USER}")
                    if(true){
                        sh ("kubectl set image deployment/django localhost:32121/${TRAINING_USER}/django-pipeline/django:${env.BUILD_NUMBER} -n ${TRAINING_USER}")
                    }
                }
                catch(e){
                
                    sh ("sed -i 's/###USER###/${TRAINING_USER}/g' deployment.yaml")
                    sh ("sed -i 's/###TAG###/${env.BUILD_NUMBER}/g' deployment.yaml")
                    sh ("kubectl apply -f deployment.yaml -n ${TRAINING_USER} ")
                    echo "deploying"
                }
                sh ("kubectl get pods -n ${TRAINING_USER}")
            }
        }
    }
 }
