---
layout: post
title: "Kubernetes Quiz(2)"
summary: Container 기초개념 quiz
author: Dohyun Kim
date: 2021-04-13 21:30:00 -0400
category: Kubernetes
thumbnail: /assets/img/posts/Kubernetes.png
comments: true
---
#### 기존 Hypervisor 가상화 vs Docker & Container 가상화
기존 방식은 HostOS 위에 GuestOS를 생성하고 그 위에서 돌기 때문에 너무 무거움. 반면 Container를 사용하면
단일 Host OS에서 여러개의 process가 각각 독립된 공간으로 인식하고 돌기 때문에 가볍고 빠르다.

격리 기술 -> namespace 와 Cgroup

**namespace** - 사용되는 다양한 변수 등의 name들을 분리해서 각 group이 독자적으로 사용하게 함. (A group B group 이 서로 분리되게 하여 같은 이름(process 1, process 2...)을 사용하더라도 논리적으로 분리돼서 괜찮음.)

**cgroup** - process group의 cpu, memory 등의 system resource를 격리.  
process가 사용하는 시스템 자원용량 제한

#### Container 의 장점
- application의 빠른 배포. (portability)(Container 이식이 편함.)
- 하나의 OS에서 동작하므로 resource들을 효율적 사용.
- MSA 구조로 SW를 짜기 쉬움. (빠른 build와 빠른 배포가 가능하므로.)
    --> 마찬가지로, 유지보수가 쉬움.

#### Container 의 단점
- 하나의 동일 Kernel을 공유하기 때문에 보안에 취약함.(하나의 container를 통해 다른 container로의 침범이 가능.)
- HostOS에 종속적이다. (Linux용 컨테이너는 Linux위에서만 돌릴 수 있음)


### Container 기초개념
Container - 돌고자 하는 program이 필요한 library, setting 해야 하는 configuration 등을 한꺼번에 표준화된 방법으로 packaging 화 해둔 것.

- Standard Packaging  
    표준화된 방법으로 prgram에 필요한 lib, config 등을 패키징화 해둠.

- Isolation and Efficiency   
    시스템 자원 사용량 격리. Container간 namespace 격리.

- Portable  
    이식성 우수. 관계없이 돌아갈 수 있음.

- Separation of Concerns   
    걱정할 거리. 가 없음.(Infra관리에 대한 걱정이 없음)


### Cloud Native
- Self-Service  
    모든 bin/lib들이 packaging화 되어 있어서 부수적으로 다른 것 없이 돌아갈 수 있음.

- API-Driven  
    MSA들의 연결--Restful API를 이용

- Elastic  
    즉 Agile 하다. 복제, 축소, 교체 등이 유연하다.

기술적인 면(Technical)
---
- DevOps : 개발과 운용을 한 team으로 구성.  
    (Infra들도 또 다른 program으로 돌아감. Software Defined Infra)
- Microservices : service를 작은 단위로 쪼개고 각각은 독립적으로 운용가능.
- CI/CD : (Continuous Integration, Continuous Deployment) 바로바로 build, 배포가 용이. 지속적 build 배포.
- Containers : library와 의존성 등을 packaging화 해서 독립적으로 운용 가능.

상업적인 면(Business)
---
- Agile : 변화, 변경에 대응하기 쉬움.
- Vendor Agnositc : vendor(특정 제품)에 종속되지 않는다. 무관하다. (AWS에서 돌리던 것을 MS Azure에서도 돌릴 수 있다. MSA구조 => 종속성이 없음.)
- CapEX(Capital expense)/OpEX(Operation expense) : 투자비와 운용비가 그다지 들지 않음.


