---
title: "워크로드"
weight: 50
description: >
  쿠버네티스에서 배포 가능한 가장 작은 컴퓨팅 오브젝트인 파드와 이를 실행하는 데 도움이 되는 상위 레벨의 추상화를 이해한다.
no_list: true
---

{{< glossary_definition term_id="workload" length="short" >}}
워크로드가 단일 구성 요소인지 아니면 함께 작동하는 여러 구성 요소인지에 관계없이,
쿠버네티스에서는 [파드](/ko/docs/concepts/workloads/pods) 집합 내에서 워크로드를 실행한다.
쿠버네티스에서 파드는 클러스터에서 실행 중인 {{< glossary_tooltip text="containers" term_id="container" >}}
집합을 나타낸다.

파드에는 수명주기가 정의되어 있다. 예를 들어, 파드가 클러스터에서 실행되면 해당 파드가 실행 중인
{{< glossary_tooltip text="노드" term_id="node" >}} 에 심각한 오류가 발생하면
해당 노드의 모든 파드가 실패한다는 것을 의미한다. 쿠버네티스는 이 정도의 수준의 실패를
최종적으로 처리한다. 나중에 노드가 복구되더라도 새 파드를 만드러야 한다.

그러나 작업을 상당히 쉽게 하기 위해 각 타드를 직접 관리할 필요는 없다.
대신  _워크로드 리소스_ 를 사용하여 파드 집합을 관리할 수 있다.
이러한 리소스는 지정한 상태와 일치하도록 올바른 종류의 파드가 실행 중인지 확인하는
{{< glossary_tooltip term_id="controller" text="컨트롤러" >}}
를 구성한다.

워크로드 리소스에는 다음이 포함된다.

* [디플로이먼트](/ko/docs/concepts/workloads/controllers/deployment/) 와 [레플리카셋](/ko/docs/concepts/workloads/controllers/replicaset/)
  (기존 리소스 {{< glossary_tooltip text="ReplicationController" term_id="replication-controller" >}} 대체);
* [스테이트풀셋](/ko/docs/concepts/workloads/controllers/statefulset/);
* [데몬셋](/ko/docs/concepts/workloads/controllers/daemonset/) 스토리지 드라이버 또는 네트워크 플러그인과 같은
  노드 로컬 기능을 제공하는 파드 실행용.
* [잡](/ko/docs/concepts/workloads/controllers/job/) 과
  [크론잡](/ko/docs/concepts/workloads/controllers/cron-jobs/)
  가 완료 될 때까지 실행되는 작업.

또한 관련성을 찾을 수 있는 두가지 지원 개념도 있다.
* [가비지(garbage) 수집](/ko/docs/concepts/workloads/controllers/garbage-collection/) 은 _소유하고 있는 리소스_ 가
  제거된 후 클러스터에서 개체를 정리한다.
* [완료된 리소스를 위한 TTL 컨트롤러](/ko/docs/concepts/workloads/controllers/ttlafterfinished/)
  는 작업이 완료된 후 정해진 시간이 지나면 잡을 제거 한다.

## {{% heading "whatsnext" %}}

각 리소스 및 리소스와 관련된 특정 작업에 대해 배울 수 있다.

* [스테이트리스 애플리케이션 디플로이먼트 실행하기](/docs/tasks/run-application/run-stateless-application-deployment/)
* 스테이트풀 애플리케이션을 [단일 인스턴스](/ko/docs/tasks/run-application/run-single-instance-stateful-application/)
  혹은 [복제된 셋](/docs/tasks/run-application/run-replicated-stateful-application/) 로 실행
* [크론잡으로 자동화된 작업 실행하기](/ko/docs/tasks/job/automated-tasks-with-cron-jobs/)

애플리케이션이 실행되면 인터넷에서 [서비스](/ko/docs/concepts/services-networking/service/)
로 사용하거나 웹 애플리케이션의 경우에만 [인그레스](/ko/docs/concepts/services-networking/ingress)
로 사용할 수 있다.

[구성](/ko/docs/concepts/configuration/) 페이지에서 코드를 분리하는 쿠버네티스
매커니즘에 대해 배울 수 있다.
