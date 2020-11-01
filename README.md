## Приветрный ответ на задание ESR

**Мысли довольно простые:**  
+ Во время простоя мимнимум затрат
+ автомасштаирование при росте нагрузки: при достижении 60% на CPU поднимаеи еще один под на ноде,  
это с учетом 5-10секунд задержки при старте.
+ И мало ли рост бужет больше стрес теста, 1 под в запасе.

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
**Сколько сразу запускать реплик**
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.9-alpine
	imagePullPolicy: Always
**Раочее кол-во ресурсов**
	resources:
          requests:
            cpu: "0.1"
	    memory: "128Mi"
**максимально потреляемое кол-во ресурсов**
          limits:
            cpu: "0.9"
	    memory: "150Mi"
        ports:
        - containerPort: 80
          protocol: TCP
**включаем проверку работоспособности, если не работает, то перезагрузить контейнер**
	livenessProbe:
            enabled: true
            type: http
**Время ожидания после запуска pod'ы до момента начала проверок**
            initialDelaySeconds: 15
	    intervalSeconds: 3
            TimeoutSeconds: 1
**http проверка**
            httpGet:
              path: /
              port: 80
**Проверяем запустилась ли машина или не забита ли она трафиком, чтобы снять или пустить на нее трафик**
	readinessProbe:
           httpGet:
             path: /
             port: 80
	   intervalSeconds: 2
           timeoutSeconds: 2
      portals:
        - destination: nginx

**livenessProbe: - надо включать осторожно зная как минимум специфику бекэнда.**


apiVersion: autoscaling/v2beta2
**Горизонтальное масшабирование исходя из загрузки по CPU от 1 до 5 контейнеров**
**+1 контейнер в поде при достижениие 60% 1CPU контенйером.**
kind: HorizontalPodAutoscaler
metadata:
 name: nginx-hpa
spec:
 minReplicas: 1
 maxReplicas: 5
 scaleTargetRef:
   apiVersion: extensions/v1beta1
   kind: Deployment
   name: nginx
 metrics:
 - type: Resource
   resource:
     name: cpu
     target:
       type: Utilization
       averageUtilization: 60

apiVersion: networkservices/v1beta1
kind: Service
metadata:
  name: nginx-load-balancer
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      name: http
```
