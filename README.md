# IBM-Cloud-Red-Hat-Openshift-expose-app VPC

Crear nuevo projecto
****![image](https://github.com/user-attachments/assets/3b4422f8-316f-4d88-8adf-4a1e5d851c9b)

![image](https://github.com/user-attachments/assets/13647ab9-e83a-40b0-aac7-896b5f151574)

Crear aplicativo 

![image](https://github.com/user-attachments/assets/ffc1ffe0-07f2-4b2b-bb39-5961dc023272)

git repository: https://github.com/nodeshift-starters/devfile-sample.git

![image](https://github.com/user-attachments/assets/90de5d8b-7e65-4a2d-905c-e6ffdebb5002)

## Route 

Crear service

- port: Es el puerto que el Service expone internamente dentro del clúster. Es el puerto al que otros servicios, pods o el Route pueden acceder para llegar a tu aplicación. Tú decides qué puerto exponer.
- targetPort: Es el puerto en el que tu aplicación realmente está escuchando dentro del pod. Es decir, el puerto en el que tu contenedor o aplicación está sirviendo.

Revisa el target port en los detalles del deployment en la consola de ibmcloud.

![image](https://github.com/user-attachments/assets/6e7c3932-f069-4f7c-af0a-24525513c817)

Entrar a la cli de ibmcloud.

Ingresamos a nuestro projyecto
'''
oc project <project_name>
'''

```
oc expose deployment <deployment_name> --name=<service_name> --port=<port_number> --target-port=<target_port_number>
```
Exponemos el service mendiante un route.
```
oc expose service <service_name>
oc expose service service-devfile-sample-git
```
En este caso la aplicicion esta expuesta por http, en caso de querer exponela por https, es necesario agregar la siguiente configuracion en el spce del yaml de nuestra route.
'''
  tls:
    termination: edge
    insecureEdgeTerminationPolicy: Allow
'''
![image](https://github.com/user-attachments/assets/8d101169-d459-4269-9853-809db482770c)


## Ingress 

Revisar el id del cluster en la consola de ibmcloud
![image](https://github.com/user-attachments/assets/21ad4f80-a2a2-4a39-ae04-30acf4463bee)

Abrir cli de ibmcloud

Verificar dominios validos del cluster 

```
ibmcloud oc nlb-dns ls --cluster <cluster-id>
```
Es posible crear subdominios personalizados a partir de los dominios de nuestro cluster

```
test.<valid-domain>
```
Ingresamos a nuestro projyecto
'''
oc project <project_name>
'''
Creamos un service para nuestra aplicacion
```
oc expose deployment <deployment_name> --name=<service_name> --port=<port_number> --target-port=<target_port_number>
```
Crear el yaml que defina nuestro ingress resource.
```
nano ingress.yaml
```
Nota: los path's anadidos deben existir en la aplicacion que se este exponiendo.
Nota: el puerto indicado en el yaml debe ser el que este escuchando el service.
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nodejs-basic-ingress
  namespace: <namespace|project>
spec:
  rules:
  - host: <valid-domain|subdomain>
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: <service_name>
            port:
              number: 3001
```

Doc: https://cloud.ibm.com/docs/openshift?topic=openshift-ingress-public-expose

## Load banlancer

Revisar el pod selector de nuestro deployment.

![image](https://github.com/user-attachments/assets/2e04cbed-5125-47cf-9aaf-84d16c299ae6)

Entrar a la cli de ibmcloud.

Ingresamos a nuestro projyecto
'''
oc project <project_name>
'''
Crear un yaml con la siguiente configuracion.
```
nano lb.yaml
```
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
Verificar que el load balancer se termine de provisionar 

![image](https://github.com/user-attachments/assets/3db735e2-95aa-4ace-9819-0d00248cb368)

Doc: 
