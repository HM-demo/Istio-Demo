pipeline {
   agent any
  
    stages
       {
          stage("Docker image build") {
            steps {
                cleanWs()
                checkout scm
                sh 'sudo docker build -t nodejs-image-new .'
            }
        }
           stage("Docker Tag&Push") {
            steps {
               withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId:'Dockercreds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                sh '''
                sudo docker login -u $USERNAME -p $PASSWORD
                sudo docker tag nodejs-image-new ${image}:${VERSION}
                sudo docker push ${image}:${VERSION}
                sed -i -e 's/IMGVERSION/'${VERSION}'/g' helloworld.yaml
                '''
               }   
            }
        }
           stage("ISTIO Deployment") {
               steps {
               sh ''' 
                      curl -L https://git.io/getLatestIstio | sh -
                      kubectl apply -f istio-0.5.1/install/kubernetes/istio.yaml
                      sleep 20
                      export PATH=$PWD/istio-0.5.1/bin:$PATH
                      kubectl get all -n istio-system
                      istioctl kube-inject -f helloworld-v2.yaml
                      kubectl apply -f helloworld-v2.yaml
                      kubectl apply -f helloworld-svc.yaml
                      istioctl replace -f route-traffic.yaml
                      sleep 60
                      INGRESS_URL=`kubectl get ingress helloworld -o jsonpath="{.status.loadBalancer.ingress[0].*}"`
                      for i in {1..20}; do curl $INGRESS_URL/hello >>tmp; done
                      cat tmp
                      '''
        }
     }
   }
}
