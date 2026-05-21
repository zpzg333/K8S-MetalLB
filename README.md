# K8S-MetalLB

> 베어메탈 Kubernetes 클러스터를 위한 LoadBalancer 구현.  
> MetalLB L2 모드를 통해 클라우드 없이 외부 IP를 K8s 서비스에 자동 할당.

---

## 개요

AWS, GCP 등 클라우드 환경과 달리, 베어메탈 Kubernetes에서는 `LoadBalancer` 타입 Service를 생성해도  
외부 IP가 자동으로 할당되지 않습니다 (`<pending>` 상태 고착).

**MetalLB**는 이 문제를 해결하는 오픈소스 솔루션입니다.  
L2(ARP) 또는 BGP 모드로 동작하며, 이 프로젝트는 **L2 모드**를 사용합니다.

## 동작 방식

```
kubectl apply -f metallb-SVC.yaml
        ↓
MetalLB Controller → IPAddressPool에서 IP 할당
        ↓
MetalLB Speaker → L2 ARP 응답으로 IP 광고
        ↓
외부 클라이언트 → 할당된 External IP로 서비스 접근
```

## 구조

```
K8S-MetalLB/
├── metallb-deploy.yaml    ← MetalLB Controller + Speaker 배포
├── metallb-daemonset.yaml ← Speaker DaemonSet
├── metallb-ippool.yaml    ← IP 주소 풀 (L2Advertisement)
├── metallb-l2-adv.yaml    ← L2 광고 정책
└── metallb-SVC.yaml       ← 예시 LoadBalancer Service
```

## 실행 방법

```bash
# MetalLB 네임스페이스 및 컴포넌트 배포
kubectl apply -f metallb-deploy.yaml
kubectl apply -f metallb-daemonset.yaml

# IP 풀 및 L2 광고 설정
kubectl apply -f metallb-ippool.yaml
kubectl apply -f metallb-l2-adv.yaml

# 서비스 배포 및 External IP 확인
kubectl apply -f metallb-SVC.yaml
kubectl get svc -A
```

## 기술 스택

| 항목 | 기술 |
|---|---|
| 플랫폼 | Kubernetes (베어메탈) |
| LoadBalancer | MetalLB |
| IP 광고 방식 | L2 (ARP) |

## 연관 프로젝트

MetalLB로 할당된 외부 IP는 아래 프로젝트에서 활용됩니다:
- [my-k8s-dns](../my-k8s-dns) — CoreDNS Service에 External IP 할당
- [Ansible](../Ansible) — Ansible 웹 서버 외부 노출

## 배경

베어메탈 홈랩 환경에서 LoadBalancer 기능을 구현하여,  
클라우드와 동일한 방식으로 Kubernetes 서비스를 외부에 노출하기 위한 프로젝트입니다.
