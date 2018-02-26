pipeline {
   agent any
   
   environment {
       image = "pavanraj29/helloworld"
      VERSION = "${BUILD_ID}"
      CurrVersion = "pre"
   }
  
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
                sed -i -e 's/IMGVERSION/'${VERSION}'/g' helloworld-deploy.yaml
                sed -i -e 's/NEWVERSION/'${VERSION}'/g' route-traffic-90-10.yaml
                sed -i -e 's/NEWVERSION/'${VERSION}'/g' route-traffic-50-50.yaml
                sed -i -e 's/NEWVERSION/'${VERSION}'/g' route-traffic-0-100.yaml
                '''
               }   
            }
        }
           stage("ISTIO Deployment") {
               steps {
               sh ''' 
                      sed -i -e 's/CURRVERSION/'${CurrVersion}'/g' route-traffic-90-10.yaml
                      sed -i -e 's/CURRVERSION/'${CurrVersion}'/g' route-traffic-50-50.yaml
                      sed -i -e 's/CURRVERSION/'${CurrVersion}'/g' route-traffic-0-100.yaml
                      curl -L https://git.io/getLatestIstio | sh -
                      kubectl apply -f istio-0.5.1/install/kubernetes/istio.yaml
                      sleep 20
                      export PATH=$PWD/istio-0.5.1/bin:$PATH
                      kubectl get all -n istio-system
                      istioctl kube-inject -f helloworld-deploy.yaml
                      kubectl apply -f helloworld-deploy.yaml
                      kubectl apply -f helloworld-svc.yaml
                      //routing 10% of traffic to new version
                      istioctl replace -f route-traffic-90-10.yaml
                      sleep 30
                      //perform tests
                      INGRESS_URL=`kubectl get ingress helloworld -o jsonpath="{.status.loadBalancer.ingress[0].*}"`
                      for i in {1..20}; do curl $INGRESS_URL/hello >>tmp; done
                      cat tmp
                      //routing traffic to 50-50
                      istioctl replace -f route-traffic-50-50.yaml
                      rm tmp
                      sleep 30
                      INGRESS_URL=`kubectl get ingress helloworld -o jsonpath="{.status.loadBalancer.ingress[0].*}"`
                      for i in {1..20}; do curl $INGRESS_URL/hello >>tmp; done
                      cat tmp
                      //routing all traffic to new version
                      istioctl replace -f route-traffic-50-50.yaml
                      rm tmp
                      sleep 30
                      INGRESS_URL=`kubectl get ingress helloworld -o jsonpath="{.status.loadBalancer.ingress[0].*}"`
                      for i in {1..20}; do curl $INGRESS_URL/hello >>tmp; done
                      cat tmp
                      //removing old version
                      kubectl delete pods -l version=v${CurrVersion}
                      '''
        }
     }
   }
}
