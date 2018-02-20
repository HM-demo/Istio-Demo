pipeline {
   agent any
  
    stages
       {
           stage("ISTIO Deployment") {
               steps {
                    cleanWs()
               sh ''' 
                      istioctl kube-inject -f samples/bookinfo/kube/bookinfo.yaml
                      kubectl apply -f samples/bookinfo/kube/bookinfo.yaml
                      sleep 60
                      ISTIO_INGRESS=$(kubectl get ingress gateway -o jsonpath="{.status.loadBalancer.ingress[0].*}")
                      for((i=1;i<=10;i+=1));do curl  -s http://$ISTIO_INGRESS/productpage >> mfile; done;
                      a=$(grep 'full stars' mfile | wc -l) && echo Number of calls to v3 of reviews service "$(($a / 2))"
                  '''
        }
     }
   }
}
