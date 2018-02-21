pipeline {
   agent any
  
    stages
       {
           stage("ISTIO Deployment") {
               steps {
                    cleanWs()
                    checkout scm
               sh ''' 
                      curl -L https://git.io/getLatestIstio | sh -
                      kubectl apply -f istio-0.5.1/install/kubernetes/istio.yaml
                      sleep 20
                      export PATH=$PWD/bin:$PATH
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
