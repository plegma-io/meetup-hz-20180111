[Source](https://blog.hazelcast.com/deploy-monitor/ "Permalink to Deploy and Monitor Hazelcast Cluster to Kubernetes - Part 1")

# Deploy and Monitor Hazelcast Cluster to Kubernetes - Part 1

[hazelcast][1] 클러스터를 [kubernetes][2] 에서 배포 하는 데모/튜토리얼 입니다.

## Prerequisite
* Minikube 를 사용 
* Kubectl 클라이언트 설치
* [설치가이드][16]

## Scenario

헤이즐캐스트 클러스터와 헤이즐캐스트 매니지먼트센터를 kubernetes 에 간편하게 배포해 보겠습니다. 또한 헤이즐캐스트 메니지먼트센터를 사용하여 헤이즐캐스트 클러스터를 모니터링 해보고 scale-out/in 을 해보려고 합니다.

지금은 어떤 pod도 실행되고 있지 않습니다. 데모는 minikube 로 실행됩니다 [minikube][3].

![enter image description here][4]

Kubernetes 에 배포하기 위한 deployment 파일을 살펴 보도록 하겠습니다. 간단히 헤이즐캐스트의 도커 이미지를 설정을 하고 2개의 레플리카를 생성 하도록 하겠습니다 [hazelcast/hazelcast][14].
    
    
    apiVersion: apps/v1beta1
        kind: Deployment
        metadata:
          name: hazelcast-deployment
        spec:
          replicas: 2
          template:
            metadata:
              labels:
                app: hazelcast
            spec:
              containers:
              - name: hazelcast
                image: hazelcast/hazelcast
    

위의 예제 `deployment.yml` 파일을 생성 해주세요 또는 깃헙에서 받아주시면 됩니다.

아래의 명령어를 실행하여 배포를 합니다:
    
    
    kubectl apply -f deployment.yml
    

아래의 명령어를 실행하여 pod의 정보를 볼 수 있습니다.
    
    
    kubectl get pods
    

![enter image description here][5]

배포에 걸리는 시간은 대략 3-5 분 정도 걸리고 배포가 완료되면 `running` 이라고 표시 됩니다.

아래의 명령어를 사용하여 각 pod의 로그를 볼 수 있습니다.
    
    
    kubectl logs $POD_NAME
        

![enter image description here][6]

로그를 보시면 헤이즐캐스트 클러스터가 생성된것을 볼 수 있습니다. overlay 네트워크를 사용하게되면 멀티캐스트로 특별한 설정없이 클러스터를 자동으로 생성할 수 있게 됩니다. ( I have tried this scenario in KOPS also, it is verified )

이젠 매니지먼트센터를 추가해 보도록 하겠습니다.

아래의 내용을 `deployment.yml` 파일에 추가해주세요
    
    
    ---
       apiVersion: apps/v1beta1
       kind: Deployment
       metadata:
         name: management-center
       spec:
         replicas: 1
         template:
           metadata:
             labels:
               app: management-center
           spec:
             containers:
             - name: hazelcast
               image: hazelcast/management-center
               

설정파일을 세이브후에 다시 실행:
    
    
    kubectl apply -f deployment.yml
        

![enter image description here][7]

`kubectl logs $POD_NAME` 실행해서 로그를 보면 메니지먼트센터가 잘 실행된것을 확인해 볼 수 있습니다. 

![enter image description here][8]

이젠 메니지먼트센터를 [services][9]로 설정하고 pod 를 expose 하여 브라우저에서 볼 수 있게 해보겠습니다.

새로운 서비스로 등록하여서 배포하겠습니다 아래의 내용을 `deployment.yml` 파일 맨 아래에 추가해 주세요.
    
    
    ---
        kind: Service
        apiVersion: v1
        metadata:
          name: my-service
        spec:
          type: NodePort
          selector:
            app: management-center
          ports:
            - protocol: TCP
              port: 8080
              targetPort: 8080
        

마찬가지로 배포 커맨드를 실행합니다.

![enter image description here][11]

minikube 커맨드를 실행하여 서비스를 실행 할 수 있습니다:
    
    
    minikube service my-service
        

브라우저가 실행되면 url 뒤에 `mancenter` 를 추가하면 메니지먼트센터로 들어갈 수 있습니다.

![enter image description here][12]

## Conclusion

여기까지 2pods 를 배포해보았고 1개의 서비스를 실행하여 메니지먼트센터를 볼 수 있었습니다.
또한 Kubernetes 커맨드를 사용하여서 레플리카를 늘리거나 줄이는것을 해 볼수 있습니다. 
물론 대쉬보드에서도 같은 작업이 가능합니다.

## What's Next

hazelcast.xml 에 커스텀 세팅을 하여 헤이즐캐스트 클러스터를 배포 하는 등의 더욱 재미난 시간이 준비되어있습니다.

---
### Kubernetes & Minikube commands
#### Kubernetes commands:
```
//show pods
kubectl get pods 
//show more info on pods
kubectl get pods --output=wide 
//show all pods including system pods
kubectl get pods --all-namespaces 

//show detail description
kubectl describe deployment/hazelcast-deployment 

//scale-out to 4 replicas
kubectl scale --replicas=4 deployment/hazelcast-deployment 

//show logs
kubectl log $POD_NAME

//delete all pods
kubectl delete --all pods --namespace=default 

//delete all deployments
kubectl delete --all deployments --namespace=default

//delete all services
kubectl delete --all services --namespace=default 
```
#### Minikube commands:
```
//start minikube with memory size set
minikube start --memory 8192 

//launch dashboard
minikube dashboard

//show status of minikube
minikube status

//shutdown minikube
minikube stop
```

[1]: https://hazelcast.org/ "Hazelcast"
[2]: https://kubernetes.io/ "Kubernetes"
[3]: https://github.com/kubernetes/minikube/releases "Releases"
[4]: https://cdn-images-1.medium.com/max/800/1*Tpne41f_32J1AttJ68ODNQ.png "kubectl get pods no resources"
[5]: https://cdn-images-1.medium.com/max/800/1*lznE5OOSXhjIUgPFNIoN9w.png "kubectl get pods"
[6]: https://cdn-images-1.medium.com/max/800/1*G4T-gJhKApNI2hWx_HYdsg.png "kubectl logs"
[7]: https://cdn-images-1.medium.com/max/800/1*r0On6L9v2g_kivy1b_yWuQ.png "kubectl apply -f"
[8]: https://cdn-images-1.medium.com/max/800/1*6Pd2-GPzekIds2GFoGQCtw.png "kubectl logs management"
[9]: https://kubernetes.io/docs/concepts/services-networking/service/ "Kubernetes Networking"
[10]: https://kubernetes.io/docs/concepts/services-networking/ingress/ "Ingress Resources"
[11]: https://cdn-images-1.medium.com/max/800/1*XS_FW0iDaAP_O_X3OEfuQQ.png "kubectl apply -f deployment"
[12]: https://cdn-images-1.medium.com/max/800/1*CAV4a7r89Rzg9ay_T-mEFg.png "Memory Utilization"
[13]: https://gist.githubusercontent.com/anonymous/ca3d46c5185f8def24d16e0877f7486b/raw/b6dd5272ce573ccf84a05bab9205bd6431780ad3/gistfile1.txt "Deployment File"
[14]: https://hub.docker.com/r/hazelcast/hazelcast/ "Docker Hub"
[15]: 
https://github.com/pires/hazelcast-kubernetes/
"Hazelcast Discovery SPI example"
[16]: https://drive.google.com/open?id=1e3qiGwLe7fr55FRagHe_162Pec0Qc16ZViZhMWkUtgw "Kubernetes client $ Minikube install guide"