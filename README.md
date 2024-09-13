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

---

## Métodos de Exposición de Aplicaciones

### Exponer Aplicación mediante Route

**Descripción**: El método más sencillo y rápido para exponer una aplicación. Crea una URL pública que enruta directamente hacia el servicio que ejecuta tu aplicación.

#### Crear Servicio

1. **port**: Es el puerto que el Service expone internamente dentro del clúster. Otros servicios, pods o Routes acceden a este puerto para conectarse a tu aplicación.
2. **targetPort**: Es el puerto donde la aplicación escucha dentro del pod. Se debe asegurar que coincida con el puerto donde tu aplicación está sirviendo peticiones.

Revisa el `targetPort` en los detalles del deployment en la consola de IBM Cloud.

![image](https://github.com/user-attachments/assets/6e7c3932-f069-4f7c-af0a-24525513c817)

#### Exponer el Deployment mediante CLI

```bash
oc project <project_name>
oc expose deployment <deployment_name> --name=<service_name> --port=<port_number> --target-port=<target_port_number>
```

#### Crear Route

Exponemos el servicio mediante un Route. Este crea una URL para que accedan los usuarios externos.

```bash
oc expose service <service_name>
```

**Aclaración**: Esta exposición es por HTTP. Si necesitas exponer la aplicación por HTTPS, agrega la siguiente configuración en el YAML del Route. HTTPS es crucial si quieres asegurar las comunicaciones con cifrado TLS.

```yaml
tls:
  termination: edge
  insecureEdgeTerminationPolicy: Redirect
```

![image](https://github.com/user-attachments/assets/8d101169-d459-4269-9853-809db482770c)

**Ventajas de Route**:
- Útil para exposiciones rápidas.
- Ideal para aplicaciones sencillas que no requieren configuraciones complejas de tráfico.
  
---

### Exponer Aplicación mediante Ingress

**Descripción**: Utilizado cuando necesitas tener un control más granular sobre las reglas de tráfico de red. Te permite definir subdominios, reglas de path y opciones avanzadas de red, ideales para aplicaciones más complejas.

#### Verificar Dominios Válidos del Clúster

Abre la CLI de IBM Cloud y revisa los dominios válidos disponibles para tu clúster:

```bash
ibmcloud oc nlb-dns ls --cluster <cluster-id>
```

**Aclaración**: Ingress te permite manejar múltiples rutas bajo un mismo dominio, lo que es útil si deseas exponer varias aplicaciones bajo diferentes paths o subdominios.

### Crear Ingress Resource

Creamos el archivo YAML para el Ingress:

```bash
nano ingress.yaml
```

**Notas**:
- Los paths añadidos deben existir en la aplicación que se esté exponiendo.
- El puerto indicado en el YAML debe ser el que está escuchando el servicio.

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

**Ventajas de Ingress**:
- Te permite gestionar aplicaciones con múltiples rutas.
- Ofrece control avanzado para la exposición de servicios bajo diferentes dominios o paths.

---

### Exponer Aplicación mediante Load Balancer

**Descripción**: Ideal para manejar grandes cantidades de tráfico. Distribuye las solicitudes de los usuarios entre diferentes réplicas de tu aplicación, mejorando la disponibilidad y escalabilidad.

#### Crear Load Balancer

Revisa el `selector` del deployment para asegurarte de que apunta a los pods correctos.

![image](https://github.com/user-attachments/assets/2e04cbed-5125-47cf-9aaf-84d16c299ae6)

Crea el archivo YAML del Load Balancer:

```bash
nano lb.yaml
```

Define un `LoadBalancer` en el archivo YAML:

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

**Ventajas de Load Balancer**:
- Excelente para aplicaciones que necesitan manejar tráfico a gran escala.
- Permite balancear las cargas entre réplicas, mejorando la disponibilidad.
  
Doc oficial: [IBM Cloud Load Balancer](https://cloud.ibm.com/docs/openshift?topic=openshift-load-balancer)

---

## Comparación de Métodos de Exposición

- **Route**: Ideal para aplicaciones pequeñas o rápidas que requieren acceso público sin necesidad de configuraciones avanzadas.
- **Ingress**: Útil para aplicaciones más complejas que requieren gestionar múltiples rutas o dominios bajo un mismo clúster.
- **Load Balancer**: La mejor opción para aplicaciones con grandes volúmenes de tráfico y que requieren alta disponibilidad y escalabilidad.
