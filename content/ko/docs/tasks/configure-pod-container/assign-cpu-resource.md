---
title: 컨테이너 및 파드에 CPU 리소스 할당
content_type: task
weight: 20
---

<!-- overview -->
이 페이지는 CPU *요청량* 과 CPU *상한* 을 컨테이너에 어떻게 할당하는
방법을 보여준다. 컨테이너는 구성된 상한보다 더 많은 CPU를 사용할 수 없다.
시스템에 사용 가능한 CPU 시간이 있는 경우, 컨테이너는 요청한 만큼의
CPU를 할당받을 수 있다.




## {{% heading "prerequisites" %}}


{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

작업 예제를 실행하려면 클러스터에 적어도 하나의 CPU를 사용할 수 있어야 한다.


이 페이지의 몇 가지 단계를 실행하려면 클러스터에서
[메트릭-서버](https://github.com/kubernetes-sigs/metrics-server)
서비스를 실행해야 한다. 메트릭 서버가 실행 중인 경우
해당 단계를 건너뛸 수 있다.

{{< glossary_tooltip term_id="minikube" >}}를 실행하는 경우
다음 명령을 실행하여 메트릭-서버를 활성화한다.

```shell
minikube addons enable metrics-server
```

메트릭-서버(또는 리소스 메트릭 API의 다른 공급자 `metrics.k8s.io`) 가
실행 중인지 확인하려면 다음의 명령을 입력한다.

```shell
kubectl get apiservices
```

리소스 메트릭 API를 사용할 수 있는 경우, 출력에
`metrics.k8s.io`에 대한 참조가 포함된다.


```
NAME
v1beta1.metrics.k8s.io
```




<!-- steps -->

## 네임스페이스 생성

이 예제에서 생성한 리소스가 나머지 클러스터와 분리되도록
{{< glossary_tooltip term_id="namespace" >}}를 생성한다.

```shell
kubectl create namespace cpu-example
```

## CPU 요청 및 CPU 제한 지정

컨테이너에 대한 CPU 요청을 지정하려면 컨테이너 리소스 메니페스트에 `resources:requests`
필드를 포함해야 한다. CPU 제한을 지정하려면 `resources:limits` 를 포함해야 한다.

이 예제에서는 한 개의 컨테이너가 있는 파드를 만든다. 컨테이너에는
0.5 CPU의 요청과 1개의 CPU 제한이 있다. 다음은 파드의 구성 파일이다.

{{< codenew file="pods/resource/cpu-request-limit.yaml" >}}

구성 파일의 `args` 섹션은 컨테이너가 시작될 때 인수를 제공한다.
`-cpus "2"` 인수는 컨테이너가 2개의 CPU를 사용하도록 한다.

파드 생성하기.

```shell
kubectl apply -f https://k8s.io/examples/pods/resource/cpu-request-limit.yaml --namespace=cpu-example
```

파드가 실행 중인지 확인하기.

```shell
kubectl get pod cpu-demo --namespace=cpu-example
```

파드에 대한 자세한 정보 보기.

```shell
kubectl get pod cpu-demo --output=yaml --namespace=cpu-example
```

출력은 파드의 한 컨테이너에 500 milliCPU의 요청과 1개의 CPU 제한이
있음을 보여준다.

```yaml
resources:
  limits:
    cpu: "1"
  requests:
    cpu: 500m
```

`kubectl top` 을 사용하여 파드에 대한 메트릭을 가져온다.

```shell
kubectl top pod cpu-demo --namespace=cpu-example
```

이 예제의 출력 결과는 파드가 974 milliCPU를 사용하고 있음을 보여준다.
이는 파드 구성에 지정된 1 CPU 제한보다 약간 적은 것이다.

```
NAME                        CPU(cores)   MEMORY(bytes)
cpu-demo                    974m         <something>
```

`-cpu "2"`를 설정하여 2개의 CPU를 사용하도록 컨테이너를 구성했지만, 컨테이너는 약 1개의 CPU만 사용하도록 허용된다는 점을 상기해야 한다. 컨테이너가 제한보다 더 많은 CPU 리소스를 사용하려고 했기 때문에 컨테이너의 CPU 사용이 제한되고 있다.

{{< note >}}
1.0 이하의 CPU 사용에 대한 또 다른 설명은 노드에 사용가능한 CPU 리소스가
충분하지 않을 수 있다는 것이다. 이 실습의 전제 조건에서는 클러스테어 사용 가능한 CPU가 1개 이상 있어야 한다는 점을 기억해야한다..
만약 컨테이너가 CPU가 1개만 있는 노드에서 실행되는 경우 컨테이너에 지정된 CPU 제한에 관계없이 둘 이상의 CPU를 사용할 수 없다..
{{< /note >}}

## CPU units

The CPU resource is measured in *CPU* units. One CPU, in Kubernetes, is equivalent to:

* 1 AWS vCPU
* 1 GCP Core
* 1 Azure vCore
* 1 Hyperthread on a bare-metal Intel processor with Hyperthreading

Fractional values are allowed. A Container that requests 0.5 CPU is guaranteed half as much
CPU as a Container that requests 1 CPU. You can use the suffix m to mean milli. For example
100m CPU, 100 milliCPU, and 0.1 CPU are all the same. Precision finer than 1m is not allowed.

CPU is always requested as an absolute quantity, never as a relative quantity; 0.1 is the same
amount of CPU on a single-core, dual-core, or 48-core machine.

Delete your Pod:

```shell
kubectl delete pod cpu-demo --namespace=cpu-example
```

## Specify a CPU request that is too big for your Nodes

CPU requests and limits are associated with Containers, but it is useful to think
of a Pod as having a CPU request and limit. The CPU request for a Pod is the sum
of the CPU requests for all the Containers in the Pod. Likewise, the CPU limit for
a Pod is the sum of the CPU limits for all the Containers in the Pod.

Pod scheduling is based on requests. A Pod is scheduled to run on a Node only if
the Node has enough CPU resources available to satisfy the Pod CPU request.

In this exercise, you create a Pod that has a CPU request so big that it exceeds
the capacity of any Node in your cluster. Here is the configuration file for a Pod
that has one Container. The Container requests 100 CPU, which is likely to exceed the
capacity of any Node in your cluster.

{{< codenew file="pods/resource/cpu-request-limit-2.yaml" >}}

Create the Pod:

```shell
kubectl apply -f https://k8s.io/examples/pods/resource/cpu-request-limit-2.yaml --namespace=cpu-example
```

View the Pod status:

```shell
kubectl get pod cpu-demo-2 --namespace=cpu-example
```

The output shows that the Pod status is Pending. That is, the Pod has not been
scheduled to run on any Node, and it will remain in the Pending state indefinitely:


```
NAME         READY     STATUS    RESTARTS   AGE
cpu-demo-2   0/1       Pending   0          7m
```

View detailed information about the Pod, including events:


```shell
kubectl describe pod cpu-demo-2 --namespace=cpu-example
```

The output shows that the Container cannot be scheduled because of insufficient
CPU resources on the Nodes:


```
Events:
  Reason                        Message
  ------                        -------
  FailedScheduling      No nodes are available that match all of the following predicates:: Insufficient cpu (3).
```

Delete your Pod:

```shell
kubectl delete pod cpu-demo-2 --namespace=cpu-example
```

## If you do not specify a CPU limit

If you do not specify a CPU limit for a Container, then one of these situations applies:

* The Container has no upper bound on the CPU resources it can use. The Container
could use all of the CPU resources available on the Node where it is running.

* The Container is running in a namespace that has a default CPU limit, and the
Container is automatically assigned the default limit. Cluster administrators can use a
[LimitRange](/docs/reference/generated/kubernetes-api/{{< param "version" >}}/#limitrange-v1-core/)
to specify a default value for the CPU limit.

## If you specify a CPU limit but do not specify a CPU request

If you specify a CPU limit for a Container but do not specify a CPU request, Kubernetes automatically
assigns a CPU request that matches the limit. Similarly, if a Container specifies its own memory limit,
but does not specify a memory request, Kubernetes automatically assigns a memory request that matches
the limit.

## Motivation for CPU requests and limits

By configuring the CPU requests and limits of the Containers that run in your
cluster, you can make efficient use of the CPU resources available on your cluster
Nodes. By keeping a Pod CPU request low, you give the Pod a good chance of being
scheduled. By having a CPU limit that is greater than the CPU request, you accomplish two things:

* The Pod can have bursts of activity where it makes use of CPU resources that happen to be available.
* The amount of CPU resources a Pod can use during a burst is limited to some reasonable amount.

## Clean up

Delete your namespace:

```shell
kubectl delete namespace cpu-example
```



## {{% heading "whatsnext" %}}



### For app developers

* [Assign Memory Resources to Containers and Pods](/docs/tasks/configure-pod-container/assign-memory-resource/)

* [Configure Quality of Service for Pods](/docs/tasks/configure-pod-container/quality-service-pod/)

### For cluster administrators

* [Configure Default Memory Requests and Limits for a Namespace](/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/)

* [Configure Default CPU Requests and Limits for a Namespace](/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/)

* [Configure Minimum and Maximum Memory Constraints for a Namespace](/docs/tasks/administer-cluster/manage-resources/memory-constraint-namespace/)

* [Configure Minimum and Maximum CPU Constraints for a Namespace](/docs/tasks/administer-cluster/manage-resources/cpu-constraint-namespace/)

* [Configure Memory and CPU Quotas for a Namespace](/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/)

* [Configure a Pod Quota for a Namespace](/docs/tasks/administer-cluster/manage-resources/quota-pod-namespace/)

* [Configure Quotas for API Objects](/docs/tasks/administer-cluster/quota-api-object/)


