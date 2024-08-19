If u don't want open the file i drop it's here. Have a nice day guys :)

apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-mindBox
spec:
  replicas: 4
  selector:
    matchLabels:
      app: web-mindBox
  template:
    metadata:
      labels:
        app: web-mindBox
    spec:
      containers:
      - name: web-app
        image: web-mindBox-repository
        resources:
          requests:
            memory: "128Mi"  
            cpu: "0.1"  
          limits:
            memory: "128Mi"  
            cpu: "1"  
        readinessProbe:  
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 10  
          periodSeconds: 5
        livenessProbe:  
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        ports:
        - containerPort: 8080
      affinity:  
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              topologyKey: "kubernetes.io/hostname"
              labelSelector:
                matchLabels:
                  app: web-mindBox

---
apiVersion: v1
kind: Service
metadata:
  name: web-app-service
spec:
  selector:
    app: web-mindBox
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
  type: ClusterIP 

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-mindBox
  minReplicas: 1 
  maxReplicas: 6
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50

---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: web-app-pdb
spec:
  minAvailable: 3
  selector:
    matchLabels:
      app: web-application
