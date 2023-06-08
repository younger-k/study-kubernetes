3주차 - ReplicaSet
==
---
ReplicaSet?
--
- [공식 문서](https://kubernetes.io/ko/docs/concepts/workloads/controllers/replicaset/)
- 파드 집합의 실행을 항상 안정적으로 유지하기 위함.
- 동일 파드 개수에 대한 가용성을 항상 보증.
  - ReplicaSet 으로 만들어진 파드 집합은 사용자가 설정한 개수를 항상 유지하려고 하기 때문에 파드를 죽여도 좀비처럼 살아남.

### 레플리카셋 만들어보기
```yaml
# replicaset-nginx.yml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: replicaset-nginx.yml
spec:
  replicas: 3
  selector: # Definition Replicaset
    matchLabels:
      app: my-nginx-pod
  template: # Definition Pods
    metadata:
      name: my-nginx-pod
      labels:
        app: my-nginx-pod
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
```

- 레플리카셋 생성
```bash
kubectl apply -f replicaset-nginx.yml
```

- 레플리카셋 조회
```bash
# 레플리카셋 조회
kubectl get rs
#NAME               DESIRED   CURRENT   READY   AGE
#replicaset-nginx   3         3         3       4m55s

# 레플리카셋 상태 조회
kubectl describe rs/replicaset-n
#ginx
#Name:         replicaset-nginx
#Namespace:    default
#Selector:     app=my-nginx-pod
#Labels:       <none>
#Annotations:  <none>
#Replicas:     3 current / 3 desired
#Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
#Pod Template:
#  Labels:  app=my-nginx-pod
#  Containers:
#   nginx:
#    Image:        nginx:latest
#    Port:         80/TCP
#    Host Port:    0/TCP
#    Environment:  <none>
#    Mounts:       <none>
#  Volumes:        <none>
#Events:
#  Type    Reason            Age    From                   Message
#  ----    ------            ----   ----                   -------
#  Normal  SuccessfulCreate  8m52s  replicaset-controller  Created pod: replicaset-nginx-r4ksr
#  Normal  SuccessfulCreate  8m52s  replicaset-controller  Created pod: replicaset-nginx-b97mw
#  Normal  SuccessfulCreate  8m52s  replicaset-controller  Created pod: replicaset-nginx-ktc7c
```

- 파드를 지워도 yml에 설정된 replicas 개수를 유지.
```bash
kubectl get pod
#NAME                     READY   STATUS    RESTARTS   AGE
#replicaset-nginx-b97mw   1/1     Running   0          16s <- 지워보자.
#replicaset-nginx-ktc7c   1/1     Running   0          16s
#replicaset-nginx-r4ksr   1/1     Running   0          16s

kubectl delete pod replicaset-nginx-b97mw
#pod "replicaset-nginx-b97mw" deleted

kubectl get pod
#NAME                     READY   STATUS    RESTARTS   AGE
#replicaset-nginx-28c44   1/1     Running   0          11s
#replicaset-nginx-ktc7c   1/1     Running   0          108s
#replicaset-nginx-r4ksr   1/1     Running   0          108s
```

- 레플리카셋 삭제
```bash
kubectl delete rs replicaset-nginx 
# replicaset.apps "replicaset-nginx" deleted
```

### 레플리카셋의 동작 원리
- yml에 지정된 selector를 기반으로 레플리카셋 개수를 유지한다.

> 만일 기존에 replicaset에 지정된 selector로 올라가있는 파드가 이미 있다면?

```yml
# my-nginx-pod.yml
apiVersion: v1
kind: Pod # api-resource 종류
metadata:
  name: my-nginx-pod
  labels:
    app: my-nginx-pod # 레플리카셋과 동일한 라벨링.
spec:
  containers:
    - name: my-nginx-container
      image: nginx:latest
      ports:
        - containerPort: 80
          protocol: TCP
```

```bash
kubectl apply -f my-nginx-pod.yml 
#pod/my-nginx-pod created

kubectl get pod
#NAME           READY   STATUS    RESTARTS   AGE
#my-nginx-pod   1/1     Running   0          6s

kubectl apply -f replicaset-nginx.yml
#replicaset.apps/replicaset-nginx created

kubectl get pod                              
#NAME                     READY   STATUS              RESTARTS   AGE
#my-nginx-pod             1/1     Running             0          36s  <- 기존 파드
#replicaset-nginx-89z8s   0/1     ContainerCreating   0          2s   <- 신규 파드
#replicaset-nginx-jzjlr   0/1     ContainerCreating   0          2s   <- 신규 파드
```

- 동일한 라벨링의 파드가 있다면, 레플리카셋의 멤버로 인식하여 동작함!
```bash
kubectl delete pods my-nginx-pod 
#pod "my-nginx-pod" deleted

kubectl get pod  
#NAME                     READY   STATUS    RESTARTS   AGE
#replicaset-nginx-89z8s   1/1     Running   0          4m6s
#replicaset-nginx-9s9kh   1/1     Running   0          9s
#replicaset-nginx-jzjlr   1/1     Running   0          4m6s
```
- 삭제하여도 동일하게 동작함.


