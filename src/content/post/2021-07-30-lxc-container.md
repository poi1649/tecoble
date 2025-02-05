---
layout: post
title: 리눅스 컨테이너에 대해 알아보자
author: [3기_파노]
tags: ['lxc']
date: "2021-07-30T12:00:00.000Z"
draft: false
image: ../teaser/linux-container.png
---


2014년, Google이 Docker를 채택해 개발자들의 주목을 끌었습니다. 2021년 현재, 이제 Docker의 존재에 대해서 모르는 사람들이 거의 없을 정도로 보편적인 기술이 되었습니다.

Docker는 태초에 LXC라는 컨테이너 기술을 기반으로 만들어진 상위 레벨의 컨테이너 기술입니다.
현재 Docker는 LXC에서 벗어나, 최적화 및 고도화한 runC라는 기술을 기반으로 하고 있습니다.

이 글에서는 Docker의 기반이었던 LXC에 대해 알아보고자 합니다.
이를 통해 컨테이너 기술이 어떻게 동작하는지 알아보겠습니다.

---

## 리눅스 컨테이너란

기존 시스템(Host OS)에서 독자적인 시스템 환경(Guest OS)을 분리, 구축하는 기술을 가상화라 합니다.

컨테이너는 기존의 시스템에 존재하는 **프로세스**를 해당 시스템에서 격리하여, 독자적인 시스템 환경을 구축하는 기술입니다. 이를 리눅스 환경에서 실제로 구현한 것이 리눅스 컨테이너입니다.

이를 가르켜 **OS수준의 가상화**라고도 합니다.

이전에는 'application 수준의 가상화' 기술이 존재했습니다. 이 기술은 기존의 OS(Host OS) 위에 Hypervisor라는 논리적 플랫폼이 필요합니다. 이 플랫폼 위에서 가상의 OS(Guset OS)를 만들어 구동 할 수 있습니다. Hypervisor가 Host OS가 Guest OS의 커널 동작을 해석해서 Host OS가 이해할 수 있는 동작으로 변환해주는 것입니다.

OS수준의 가상화가 application 수준의 가상화와 주요하게 다른 점은 컨테이너는 Host OS와 커널을 공유한다는 것입니다. 그래서 OS수준의 가상화는 Hypervisor 없이 바로 독자적인 시스템환경을 구축할 수 있습니다.

다른 말로 컨테이너로 가상화된 프로세스들은 모두 Host OS를 호환해야 합니다.

---

## 리눅스 컨테이너의 장점


리눅스 컨테이너를 사용해 얻을 수 있는 장점으로는,

1. 사용이 편리합니다.
   application 수준의 가상화는 가상 OS를 처음부터 세팅 해주어야 하지만 리눅스 컨테이너는 목표하는 어플리케이션 환경만 만들면 되기에 비교적 간편하게 가상화를 이룰 수 있습니다.
1. 작은 용량을 사용합니다.
    가상화를 하기 위해서 OS를 처음부터 새로 설치하지 않아도 됩니다. 이용하고자 하는 어플리케이션 환경만 따로 간편하게 구축을 할 수 있습니다. 따라서, 구축에 필요한 전체적인 저장 공간도 절약할 수 있습니다.
2. 상대적으로 빠릅니다.
    리눅스 컨테이너는 상대적으로 가볍습니다. Hypervisor를 이용해 가상화를 하지 않기 때문에, Hypervisor가 커널동작을 해석하는데 필요한 오버헤드가 발생하지 않습니다. 프로세스가 동작에 필요한 자원만큼만 사용합니다.

컨테이너를 통해 우리는 실제 개발환경에서 무겁고 느린 가상화를 사용하지 않아도 손쉽고 빠르게 개발을 진행할 수 있습니다.

---

# 컨테이너가 사용하는 기술

리눅스 컨테이너라는 기능이 가능할 수 있었던 이유는 리눅스 커널에서 제공하는 기능 덕분이었습니다.
리눅스 컨테이너 공식문서에서는 리눅스 컨테이너를 가능케한 기술을 설명합니다.

· namespace
· Chroot
· CGroup

이외에도 리눅스의 여러 기능이 리눅스 컨테이너를 가능케 했지만, 리눅스 컨테이너를 구성하는 가장 주요한 것은 위 세 개의 기능입니다.
namespace와 chroots, cgroups에 대해 알아보겠습니다.

-   **Chroot**
    프로세스의 루트 디렉토리를 변경하는 것입니다. 프로세스의 루트 디렉토리를 변경하게 되면, 해당 프로세스는 변경된 루트 디렉토리 바깥의 디렉토리에 접근할 수 없게 됩니다. 이를 통해 우리는 프로세스를 물리적으로 격리할 수 있습니다. 이렇게 변경된 환경을 chroot 감옥(chroot jail)이라고 합니다.

-   **namespace**
   직역하면 이름공간, 의역하면 소속이라 해석할 수 있습니다. 커널의 자원들을 나눠서 이를 특정 프로세스에 제공합니다. 이를 통해 해당 프로세스에서만 사용할 수 있는 자원으로 가상화합니다. 프로세스 테이블(PIDs), ipc, 네트워크(IP주소, 방화벽 등), User ID, 마운트 포인트 등을 가상화할 수 있습니다.

-   **Cgroups**
   프로세스의 자원을 격리하고 사용을 제어하는 기능입니다. 대표적으로 메모리, CPU자원, 네트워크, 디스크 입출력을 제어합니다.
   Cgroups의 역할은 아래와 같습니다.

      a. 자원 제한 : 특정 프로세스의 메모리 사용량을 제한.
      b. 우선 순위 : 특정 프로세스에 더 많은 CPU와 디스크 I/O 처리량을 할당.
      c. 기록 : 프로세스가 자원을 얼마나 사용하고 있는지 측정.
      d. 제어 : 프로세스를 멈추거나, 프로세스의 체크포인트를 설정해주거나 재시작 가능. 

# 마치며..

이로써 우리는 리눅스 컨테이너에 대해 대략적인 이해를 할 수 있게 되었습니다. LXC가 도커의 기반기술이었던 만큼, 향후에 DOCKER를 이해하는데 있어 많은 도움이 될 것입니다.


## Reference

- https://linuxcontainers.org/
- https://wiki.archlinux.org/title/chroot
- https://en.wikipedia.org/wiki/Linux_namespaces
- https://en.wikipedia.org/wiki/Cgroups
