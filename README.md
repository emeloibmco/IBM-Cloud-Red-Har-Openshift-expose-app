# IBM-Cloud-Red-Hat-Openshift-expose-app

## Route 
## Ingress 
## Load banlancer
```
apiVersion: v1
kind: Service
metadata:
  name: nodejsbasic-vpc-alb-dal-1
  namespace: <namespace|project>
  annotations:
    service.beta.kubernetes.io/ibm-load-balancer-cloud-provider-enable-features: '{"public": "true"}'
spec:
  type: LoadBalancer
  selector:
    <selector-key>: <selector-value>
  ports:
    - protocol: TCP
      port: 80      
      targetPort: <target port>  
  externalTrafficPolicy: Cluster 
```
```
oc apply -f lb.yaml 
```
