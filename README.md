# Exponer Aplicaciones en OpenShift (IBM Cloud VPC)

## Crear Nuevo Proyecto

Inicia creando un nuevo proyecto en tu clúster OpenShift.

![image](https://github.com/user-attachments/assets/3b4422f8-316f-4d88-8adf-4a1e5d851c9b)

![image](https://github.com/user-attachments/assets/13647ab9-e83a-40b0-aac7-896b5f151574)

## Crear Aplicativo

El siguiente paso es crear un aplicativo. Puedes clonar un repositorio de ejemplo como:

```bash
git clone https://github.com/nodeshift-starters/devfile-sample.git
```

![image](https://github.com/user-attachments/assets/90de5d8b-7e65-4a2d-905c-e6ffdebb5002)

## Exponer Aplicación mediante Route

### Crear Servicio

1. **port**: Puerto que el Service expone internamente dentro del clúster, al que otros servicios, pods o el Route pueden acceder.
2. **targetPort**: Puerto en el que tu aplicación está escuchando dentro del pod.

Revisa el `targetPort` en los detalles del deployment en la consola de IBM Cloud.

![image](https://github.com/user-attachments/assets/6e7c3932-f069-4f7c-af0a-24525513c817)

### Exponer el Deployment mediante CLI

```bash
oc project <project_name>
oc expose deployment <deployment_name> --name=<service_name> --port=<port_number> --target-port=<target_port_number>
```

### Crear Route

Exponemos el servicio mediante un Route.

```bash
oc expose service <service_name>
```

Para exposición por HTTPS, añade la siguiente configuración en el YAML del Route:

```yaml
tls:
  termination: edge
  insecureEdgeTerminationPolicy: Allow
```

![image](https://github.com/user-attachments/assets/8d101169-d459-4269-9853-809db482770c)

## Exponer Aplicación mediante Ingress

### Verificar Dominios Válidos del Clúster

Abre la CLI de IBM Cloud y revisa los dominios válidos:

```bash
ibmcloud oc nlb-dns ls --cluster <cluster-id>
```

Es posible crear subdominios personalizados a partir de estos dominios.

```bash
test.<valid-domain>
```

### Crear Ingress Resource

Creamos el archivo YAML para el Ingress:

```bash
nano ingress.yaml
```

Asegúrate de que los paths existan en la aplicación y que el puerto coincida con el servicio.

```yaml
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

Doc oficial: [IBM Cloud Ingress](https://cloud.ibm.com/docs/openshift?topic=openshift-ingress-public-expose)

## Exponer Aplicación mediante Load Balancer

### Crear Load Balancer

Revisa el `selector` del deployment.

![image](https://github.com/user-attachments/assets/2e04cbed-5125-47cf-9aaf-84d16c299ae6)

Crea el archivo YAML del Load Balancer:

```bash
nano lb.yaml
```

```yaml
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

Aplica la configuración:

```bash
oc apply -f lb.yaml
```

Verifica el provisionamiento del Load Balancer.

![image](https://github.com/user-attachments/assets/3db735e2-95aa-4ace-9819-0d00248cb368)

Doc oficial: [IBM Cloud Load Balancer](https://cloud.ibm.com/docs/openshift?topic=openshift-load-balancer)
