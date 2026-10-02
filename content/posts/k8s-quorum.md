+++
date = '2026-10-01T17:44:54+09:00'
draft = true
title = 'K8s Quorum'
+++

2노드 Proxmox 클러스터에서 quorum 교착을 풀고,
그 위에 kubeadm으로 k8s를 올린 기록.

## 문제

desktop을 WOL로 깨우는 구조라 평소 꺼져 있는데,
2노드 클러스터는 한 쪽이 없으면 quorum을 잃는다.

## 해결

`two_node: 1` + `wait_for_all: 0`
