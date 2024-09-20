# Demo <!-- omit in toc -->

# 1. Estrategia de Actualización.
## 1.1. Crear el archivo deploy-lifecycle.yaml

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: deploy-lifecycle
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
  selector:
    matchLabels:
      app: lifecycle

  template:
    metadata:
      name: lifecycle-pod
      labels:
        app: lifecycle
    spec:
      containers:
        - name: color
          image: docker.io/kodekloud/webapp-color:v1
```
> La estrategia tipo rolling update permite cambios de version del contenedor sin perder tráfico.


## 1.2. svc-lifecycle.yaml
```yaml
apiVersion: v1
kind: Service
metadata:
  name: lifecycle-svc
spec:
  type: NodePort
  ports:
    - targetPort: 8080
      port: 80
      nodePort: 30007  # Optional: specify a port in the range 30000-32767
  selector:
    app: lifecycle

```


```vim
kubectl apply -f deploy-lifecycle.yaml
kubectl apply -f svc-lifecycle.yaml
```

## 1.3. Ejecutar el siguiente comando para minikube
```vim
minikube service lifecycle-svc --url
```
> Utilizar el enlace del resultado para acceder al servicio desde el navegador

## 1.5. Listar los pods
```vim
kubectl get pods --selector=app=lifecycle
```

## 1.6. En otra terminal "Split", actualizar el Deploy
```vim
kubectl set image deployment/deploy-lifecycle color=docker.io/kodekloud/webapp-color:v2
```
> La versión de la imagen se está actualizando a :v2

## 1.7. Comprobar el Rolling-Update
```
kubectl rollout status deployment/deploy-lifecycle
```

Comprobar los pods:
```
kubectl get pods --selector=app=lifecycle
```

> el ***Status*** de los Pods varia cuando se actualizan.

> Se puede obtener los pods varias veces para ir viendo el progreso.
~~~~
NAME                                READY   STATUS              RESTARTS   AGE
deploy-lifecycle-59c74cff7b-rr8nm   1/1     Running             0          3m18s
deploy-lifecycle-59c74cff7b-pvfx4   1/1     Terminating         0          3m17s
deploy-lifecycle-8565869dbf-ngkqk   1/1     Running             0          3s
deploy-lifecycle-8565869dbf-l5txb   1/1     Running             0          3s
deploy-lifecycle-59c74cff7b-hfvg2   1/1     Terminating         0          3m21s
deploy-lifecycle-8565869dbf-wttmg   0/1     ContainerCreating   0          0s
deploy-lifecycle-59c74cff7b-tgvz9   1/1     Terminating         0          3m21s
deploy-lifecycle-8565869dbf-vd6dt   0/1     Pending             0          0s
~~~~

## 1.8. Probar la nueva versión en browser.

## 1.9. Volver a la versión anterior.
```
kubectl rollout undo deployment/deploy-lifecycle
```

## 1.10. Comprobar el Rollback
```
kubectl rollout status deployment/deploy-lifecycle
```

## 1.11. Validar cambios en pods y navegador
