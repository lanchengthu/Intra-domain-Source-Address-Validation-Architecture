---
title: Intra-domain Source Address Validation Architecture
abbrev: Intra-domain SAV Architecture
docname: draft-ietf-savnet-intra-domain-architecture-04
obsoletes:
updates:
date:
category: info
submissionType: IETF

ipr: trust200902
area: Routing
workgroup: SAVNET
keyword: SAV

author:
 -
  ins: D. Li
  name: Dan Li
  organization: Tsinghua University
  email: tolidan@tsinghua.edu.cn
  city: Beijing
  country: China
 -
  ins: J. Wu
  name: Jianping Wu
  organization: Tsinghua University
  email: jianping@cernet.edu.cn
  city: Beijing
  country: China
 -
  ins: L. Qin
  name: Lancheng Qin
  organization: Zhongguancun Laboratory
  email: qinlc@mail.zgclab.edu.cn
  city: Beijing
  country: China
 -
  ins: N. Geng
  name: Nan Geng
  organization: Huawei
  email: gengnan@huawei.com
  city: Beijing
  country: China
 -
  ins: L. Chen
  name: Li Chen
  organization: Zhongguancun Laboratory
  email: lichen@zgclab.edu.cn
  city: Beijing
  country: China



normative:
  I-D.ietf-savnet-intra-domain-problem-statement:

informative:
  RFC2827:
  RFC3704:
  I-D.ietf-savnet-general-sav-capabilities:
...

--- abstract

This document describes a generic architecture for intra-domain Source Address Validation (SAV). It provides a common framework for developing new intra-domain SAV mechanisms and describes the conditions under which such mechanisms can improve SAV accuracy with respect to existing intra-domain SAV mechanisms.

--- middle

# Introduction {#sec-intro}

Autonomous System (AS) operators can adopt Source Address Validation (SAV) on routers to detect and mitigate data packets with spoofed source addresses. Intra-domain SAV is typically applied at external interfaces of routers facing entities that are not deployed as neighboring ASes, such as a single host, a set of hosts, or a customer network with no AS (see [I-D.ietf-savnet-intra-domain-problem-statement]). Such an entity may send data packets whose source addresses are outside the source address space that the entity is authorized to use for sourcing traffic. The core task of an intra-domain SAV mechanism is to determine the source address space that each such entity is authorized to use for sourcing traffic, and to validate the source address of each incoming packet against the source address space.

Existing intra-domain SAV mechanisms, such as [RFC2827] and [RFC3704], have limitations in SAV accuracy or operational overhead, as described in [I-D.ietf-savnet-intra-domain-problem-statement]. To help address these limitations, this document describes an architecture for intra-domain SAV. This architecture focuses on data packets with spoofed source addresses that arrive at external interfaces of routers where intra-domain SAV is applied. SAV at external interfaces facing a neighboring AS, as well as SAV at internal interfaces, is outside the scope of this architecture. This architecture assumes that intra-domain routers are trusted and operate correctly. A compromised intra-domain router is outside the threat model of this architecture.

The architecture is designed to be generally applicable and extensible. It does not specify a particular mechanism, protocol, or algorithm, and does not assume pervasive deployment in an AS. Instead, it provides a basis for describing the conditions under which the architecture can improve SAV accuracy with respect to existing intra-domain SAV mechanisms. The reader is expected to be familiar with [I-D.ietf-savnet-intra-domain-problem-statement].

## Requirements Language

{::boilerplate bcp14-tagged}

# Terminology

{:vspace}
SAV Rule:
: The rule that indicates the validity of a specific source IP address or source IP prefix per router interface. It is used by a router to make SAV decisions.

SAV-specific Information:
: Information specialized for SAV rule generation. SAV-specific information may be exchanged among routers within an AS, or may be provided by an operator-managed system operated by the AS operator. It can describe the source address space that an attached entity, such as a single host, a set of hosts, or a customer network with no AS, is authorized to use for sourcing traffic.

SAV-specific Information Delivery Mechanism:
: A mechanism by which SAV-specific information is made available to a SAV Agent when the information is not locally available to that SAV Agent. Such a mechanism may be realized by reusing an existing mechanism/protocol, extending an existing mechanism/protocol, or defining a new mechanism/protocol.

SAV Agent:
: A logical function that obtains information used for SAV rule generation, generates SAV rules, and provides the generated SAV rules to routers for application at specific external interfaces. A SAV Agent may be implemented on a router or in another system. This document does not require a particular deployment location for the SAV Agent. The information used by a SAV Agent includes SAV-specific information and may also include routing information, depending on the mechanism.

SAV Information Base:
: A conceptual data store maintained by a SAV Agent. It contains information used for SAV rule generation.

# Architecture

## Overview {#sec-arch-overview}

{{fig-arch}} illustrates the conceptual components and information flow of the intra-domain SAV architecture. The architecture centers on a SAV Agent, which is a logical function that obtains information used for SAV rule generation, generates SAV rules, and provides the generated SAV rules to routers for application at specific external interfaces. A SAV Agent may be implemented on a router or in another system. This document does not require a particular deployment location for the SAV Agent.

A SAV Agent can obtain information used for SAV rule generation from information sources within the AS, including routers within the AS and operator-managed systems. Such information includes SAV-specific information and routing information. A SAV Agent processes the available information and generates SAV rules. The generated SAV rules are made available to routers for application at the corresponding external interfaces. Data-plane SAV enforcement is performed by routers at those external interfaces by validating incoming packets against those SAV rules and applying the configured traffic handling policy to packets classified as invalid, as described in [I-D.ietf-savnet-general-sav-capabilities].

~~~
+--------------------------------------+
|         Information Sources          |
|                                      |
| +----------------+  +--------------+ |
| | Routers within |  | Operator-    | |
| | the AS         |  | managed      | |
| |                |  | System       | |
| +----------------+  +--------------+ |
+------------------+-------------------+
                   |
                   | SAV-specific information
                   | and routing information
                   v
+------------------+------------------+
|              SAV Agent              |
+------------------+------------------+
                   |
                   | SAV rules
                   v
+------------------+------------------+
|    Specific external interfaces     |
|    of routers where intra-          |
|    domain SAV is applied            |
+-------------------------------------+
~~~
{: #fig-arch title="Conceptual components and information flow of the intra-domain SAV architecture."}

## Information Used for SAV Rule Generation {#sec-arch-information}

A SAV Agent uses information available to it to generate SAV rules. For intra-domain SAV, such information includes SAV-specific information and routing information.

### Routing Information

Routing information includes information in RIBs or FIBs on routers within the AS, as well as information locally configured by the AS operator to populate or interpret routing and forwarding state. Such configuration information may include prefix allocation, route origination configuration, or other routing-related configuration provided by the AS operator.

Routing information expresses reachability and forwarding behavior. For example, it can indicate which destination prefixes are reachable through a particular interface, or which prefixes are originated by a customer network of the AS operator.

An intra-domain SAV mechanism may use routing information as an input for SAV rule generation. For example, routing prefixes associated with an attached entity may be used as part of the source address space that the entity is able to use for sourcing traffic. However, routing information does not always directly or completely represent source-address authorization, especially in asymmetric routing or hidden-prefix scenarios (see [I-D.ietf-savnet-intra-domain-problem-statement]).

### SAV-specific Information {#sec-sav-specific-information}

SAV-specific information is information dedicated to SAV rule generation. It describes source-address authorization information that cannot be directly or completely inferred from routing information alone. Such information is expected to indicate the source address space that an attached entity is authorized to use for sourcing traffic.

SAV-specific information can be provided by routers within the AS. A router may have local information about an attached entity, such as the external interface facing the entity, the local attachment context, or source-prefix information associated with that attachment. When the same entity is attached to multiple external interfaces of routers within the AS, SAV-specific information from these routers can help a SAV Agent identify that these interfaces are associated with the same entity and should be considered in the same source-address authorization context. Such information is useful in asymmetric-route scenarios.

SAV-specific information can also be provided by operator-managed systems within the AS. Such systems may provide SAV-specific information that is not available from routers alone, including source prefixes that are not advertised to the AS but are authorized for use as source addresses by an attached entity. Such information is useful in hidden-prefix scenarios.

A SAV Agent may use SAV-specific information obtained from routers, operator-managed systems, or both, depending on the mechanism.

## SAV-specific Information Delivery {#sec-info-delivery}

A SAV-specific information delivery mechanism makes SAV-specific information available to a SAV Agent when the information is not locally available to that SAV Agent. The information provider and the information receiver need to support a common delivery mechanism. Such a mechanism may be realized by reusing an existing mechanism/protocol, extending an existing mechanism/protocol, or defining a new mechanism/protocol.

A solution that defines a SAV-specific information delivery mechanism should define the information semantics, encoding, transport or management method, update behavior, and error handling appropriate for that mechanism. When SAV-specific information changes, the SAV Agent (or information receiver) needs to be able to obtain the updated information in a timely manner so that the generated SAV rules can remain consistent with the latest state.

The SAV-specific information delivery mechanism should provide appropriate session security protection. This includes authentication of the communicating entities and integrity protection for the delivered information. When SAV-specific information contains private or sensitive operational information, the delivery mechanism also needs to provide confidentiality protection or otherwise prevent disclosure to unauthorized entities.

## SAV Agent and SAV Information Base {#sec-arch-agent}

A SAV Agent is the logical function responsible for obtaining information used for SAV rule generation, maintaining a SAV Information Base, and generating SAV rules.  A SAV Agent may be implemented on a router or in another system.

{{fig-sav-agent}} illustrates the conceptual processing model of a SAV Agent.  The SAV Agent obtains SAV-specific information and routing information from routers and operator-managed systems within the AS. The SAV Agent maintains the SAV Information Base and uses it to generate SAV rules.

The SAV Information Base is a conceptual data store maintained by a SAV Agent.  It contains information used for SAV rule generation.  It may also contain metadata needed by the SAV Agent to interpret or select information for rule generation, such as information source or freshness. 

~~~
+-----------------------------------------------+
|                   SAV Agent                   |
|                                               |
| SAV-specific information  Routing information |
|            +                      +           |
|            |                      |           |
|            |                      |           |
|            v                      v           |
| +-------------------------------------------+ |
| |           SAV Information Base            | |
| +---------------------+---------------------+ |
|                       |                       |
|                       v                       |
|               SAV Rule Generator              |
|                       |                       |
|                       v                       |
|                    SAV Rules                  |
+-----------------------------------------------+
~~~
{: #fig-sav-agent title="Conceptual SAV Agent processing model"}

## SAV Rule Generation {#sec-rule-generation}

A SAV Agent generates SAV rules based on information available in the SAV Information Base.  The rule-generation algorithm is mechanism-specific and is not defined by this document.

The generated SAV rules identify whether a source address or source prefix is valid for a particular external interface. Any new intra-domain SAV mechanism needs to define how the SAV Agent uses SAV-specific information and routing information for SAV rule generation.  It also needs to define how the SAV Agent handles missing, stale, or conflicting information according to the mechanism design and operator policy.

## SAV Rule Installation and Data-plane Enforcement {#sec-data-plane-enforcement}

After SAV rules are generated by a SAV Agent, they are installed on routers that apply intra-domain SAV at the corresponding external interfaces. Data-plane SAV enforcement is the process of validating incoming packets against the installed SAV rules and applying the configured traffic handling policy to packets classified as invalid. 

The action for packets classified as invalid is a matter of local policy and deployment stage.  A deployment can initially use monitoring, logging, sampling, rate-limiting, redirecting, or other conservative actions before enabling strict dropping.  Further considerations for data-plane SAV capabilities are described in {{I-D.ietf-savnet-general-sav-capabilities}}.

# Conditions for Improvement over Existing SAV Mechanisms

The architecture can improve SAV accuracy with respect to existing mechanisms when a SAV Agent can use SAV-specific information to complement routing information for SAV rule generation. Routing information can indicate reachability and forwarding behavior, but it does not always directly or completely represent the source address space that an attached entity is authorized to use for sourcing traffic. Therefore, improvement depends on whether the required SAV-specific information is available to provide information beyond the routing information available within the AS, as described in {{sec-sav-specific-information}}.

In particular, SAV-specific information can help a SAV Agent more completely determine the source address space authorized for an attached entity. For example, SAV-specific information may indicate that the same entity is attached to multiple external interfaces of routers within the AS, or that the entity is authorized to use source prefixes that are not advertised to the AS as routing information. Such information can improve SAV rule accuracy in asymmetric-route or hidden-prefix scenarios, and can reduce or avoid improper blocking of legitimate traffic and improper permitting of traffic with spoofed source addresses.

In addition, SAV-specific information needs to be authoritative or trusted according to the trust model and operational policy of the AS operator. The architecture can also reduce operational overhead with respect to manually configured filtering when SAV-specific information can be automatically obtained, updated, and used for SAV rule generation. SAV-specific information delivery needs to support automatic updates when the information at the corresponding information source changes, so that a SAV Agent can learn the latest SAV-specific information in a timely manner.

# Operational Considerations

## Incremental Deployment {#sec-incremental-deployment}

Incremental deployment can occur in two different aspects: the incremental availability of SAV-specific information, and the incremental application of SAV rules at routers.

Incremental availability of SAV-specific information means that the SAV-specific information required by a mechanism may be available for some external interfaces or attached entities, but unavailable for others. In such cases, a SAV Agent may be able to determine the source address space authorized for some attached entities, but may be unable to do so for others. When the required SAV-specific information is unavailable for some external interfaces or attached entities, operators are encouraged to use conservative traffic handling policies. Such policies may include logging, monitoring, rate-limiting, or other non-dropping actions before strict blocking is enabled.

Incremental application of SAV rules means that SAV rules are applied only at selected routers or external interfaces. This can occur due to phased deployment plans, multi-vendor environments, operational risk management, or differences in device capability. Such deployment can still provide protection at the interfaces where SAV rules are applied, without requiring pervasive deployment across all routers in an AS.

Operators should consider deployment consistency when the same entity is attached to multiple external interfaces of routers in the AS. For example, if the entity is authorized to use a particular source address space, SAV rules for that source address space may need to be applied at all relevant external interfaces facing that entity. If SAV rules are applied only at a subset of these interfaces, traffic with spoofed source addresses may still enter the AS through interfaces where SAV rules are not applied.

## Operational Visibility and Troubleshooting {#sec-operational-visibility}

Operational visibility is important for deploying and maintaining intra-domain SAV. Operators need to be able to observe validation results and traffic handling outcomes at external interfaces where intra-domain SAV is applied. This helps operators assess whether incoming packets are classified as expected and whether packets classified as invalid are handled according to the configured traffic handling policy. Operators should use telemetry mechanisms to monitor the behavior of intra-domain SAV and collect relevant information for further analysis.

# Security and Privacy Considerations

The security and privacy properties of a specific intra-domain SAV mechanism depend on how the information used for SAV rule generation is obtained, delivered, stored, and used.

Information used for SAV rule generation needs to be authoritative or trusted within the AS. A SAV Agent needs to use such information according to the trust model and operational policy of the AS operator.

When SAV-specific information is delivered to a SAV Agent, the delivery mechanism needs to provide session security protection. This includes authentication of the communicating entities and integrity protection for the delivered information. These protections are needed to ensure that the SAV Agent can verify the source of the information and that the information is not modified during delivery.

SAV-specific information may contain private or sensitive operational information. Therefore, delivery of SAV-specific information needs to be controlled within the AS and must not disclose such information outside the AS unless explicitly authorized by the AS operator. A SAV Information Base stores information used for SAV rule generation. Access to the SAV Information Base needs to be controlled so that only authorized components or operators can read or modify the stored information. 

# IANA Considerations

This document has no IANA requirements.

# Contributors

 Mingqing Huang

 Email: huangmq@vip.sina.com
 

 Fang Gao

 Email: fredagao520@sina.com

# Acknowledgements

Many thanks to the valuable comments from: Igor Lubashev, Alvaro Retana, Aijun Wang, Joel Halpern, Jared Mauch, Kotikalapudi Sriram, Rüdiger Volk, Jeffrey Haas, Xiangqing Chang, Changwang Lin, Xueyan Song, etc.

--- back

