title: network policy
author: Wtli
tags:
  - K8S
categories:
  - 后端
date: 2020-10-15 15:20:00
---
NetworkPolicy网络策略。  
网络策略（NetworkPolicy）是一种关于 Pod 间及与其他网络端点间所允许的通信规则的规范。  
NetworkPolicy 资源使用 标签 选择 Pod，并定义选定 Pod 所允许的通信规则。  
<!-- more -->

### 示例
下面是一个 NetworkPolicy 的示例:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    ports:
    - protocol: TCP
      port: 5978
```

- apiVersion: 定义了该对象表示形式的版本控制架构。服务器应将已识别的架构转换为最新的内部值，并可能拒绝无法识别的值。

- kind: 一个字符串值，表示此对象表示的REST资源。服务器可以从客户端向其提交请求的端点推断出这一点。无法更新。

- metadata: 标准对象的元数据。

- spec: 指定此NetworkPolicy所需的行为。

### SPEC

- **egress**: 每个 NetworkPolicy 可包含一个 egress 规则的白名单列表。每个规则都允许匹配 to 和 port 部分的流量。该示例策略包含一条规则，该规则将单个端口上的流量匹配到 10.0.0.0/24 中的任何目的地。
- **ingress**: 每个 NetworkPolicy 可包含一个 ingress 规则的白名单列表。每个规则都允许同时匹配 from 和 ports 部分的流量。示例策略中包含一条简单的规则： 它匹配一个单一的端口，来自三个来源中的一个， 第一个通过 ipBlock 指定，第二个通过 namespaceSelector 指定，第三个通过 podSelector 指定。
- **podSelector**: 每个 NetworkPolicy 都包括一个 podSelector ，它对该策略所应用的一组 Pod 进行选择。示例中的策略选择带有 "role=db" 标签的 Pod。空的 podSelector 选择命名空间下的所有 Pod。
- **policyTypes**: 每个 NetworkPolicy 都包含一个 policyTypes 列表，其中包含 Ingress 或 Egress 或两者兼具。policyTypes 字段表示给定的策略是否应用于进入所选 Pod 的入口流量或者来自所选 Pod 的出口流量，或两者兼有。如果 NetworkPolicy 未指定 policyTypes 则默认情况下始终设置 Ingress，如果 NetworkPolicy 有任何出口规则的话则设置 Egress。

#### EGRESS

- ports::array => 进行通信的目的端口。如果为空，所有端口都能使用，如果包含最少一条记录，将所有通信匹配端口。
> port: 给定协议上的端口。这可以是一个数字端口，也可以是一个pod上的命名端口。如果没有提供此字段，则匹配所有端口名称和编号。  
> protocol: 流量必须匹配的协议(TCP、UDP或SCTP)。如果未指定，此字段默认为TCP。

- to::array => 规定进行通信的pod。
> ipBlock:  IPBlock定义了特定IPBlock上的策略。如果设置了该字段，那么其他字段都不能设置。  \[cidr:  CIDR是一个表示IP块的字符串，有效的例子是“192.168.1.1/24”或“2001:db9::/64”;  
except:有效的例子是“192.168.1.1/24”或“2001:db9::/64”，除非值在CIDR范围之外，否则将被拒绝 )]  
> namespaceSelector:  使用集群范围的标签选择名称空间。该字段遵循标准的标签选择器语义;如果存在但为空，则选择所有名称空间。如果PodSelector也被设置，那么NetworkPolicyPeer作为一个整体在NamespaceSelector选择的名称空间中选择与PodSelector匹配的Pods。否则，它选择NamespaceSelector选择的名称空间中的所有Pods。
> podSelector:  这是一个pod选择器。该字段遵循标准的标签选择器语义;如果存在但为空，则选择所有pods。如果也设置了NamespaceSelector，那么NetworkPolicyPeer作为一个整体在NamespaceSelector选择的名称空间中选择与PodSelector匹配的Pods。否则，它在策略自己的名称空间中选择与PodSelector匹配的Pods。

#### INGRESS

- ports: 可访问的端口列表。此列表中的每一项都使用逻辑OR组合。如果该字段为空或丢失，则此规则匹配所有端口(不受端口限制的流量)。如果存在此字段并包含至少一个项，则此规则仅允许流量匹配列表中至少一个端口。

- from: 能够访问为该规则选择的pods的源列表。此列表中的项使用逻辑或操作组合。如果该字段为空或丢失，则此规则匹配所有源(不受源限制的流量)。如果存在此字段并包含至少一个项，则此规则仅允许流量匹配from列表中的至少一个项。



### 参考文档
[K8S接口文档](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#networkpolicy-v1-networking-k8s-io)  
[K8S](https://kubernetes.io/zh/docs/concepts/services-networking/network-policies/)








