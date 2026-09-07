---
title: Intra-domain Source Address Validation Architecture
abbrev: Intra-domain SAV Architecture
docname: draft-ietf-savnet-intra-domain-architecture-05
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

This document describes a generic architecture for intra-domain Source Address Validation (SAV). It provides a common framework for developing new intra-domain SAV mechanisms and describes the conditions under which this architecture can improve SAV accuracy and operational efficiency with respect to existing intra-domain SAV mechanisms.

--- middle

# Introduction {#sec-intro}

Autonomous System (AS) operators can adopt Source Address Validation (SAV) on routers to detect and mitigate data packets with spoofed source addresses. Intra-domain SAV is typically applied at external interfaces of routers facing entities that are not deployed as neighboring ASes, such as a single host, a set of hosts, or a customer network with no AS (see [I-D.ietf-savnet-intra-domain-problem-statement]). Such an entity may send data packets whose source addresses are outside the source address space that the entity is authorized to use for sourcing traffic. The core task of an intra-domain SAV mechanism is to determine the source address space that each such entity is authorized to use for sourcing traffic, and to validate the source address of each incoming packet against the source address space.

Existing intra-domain SAV mechanisms, such as [RFC2827] and [RFC3704], have limitations in SAV accuracy or operational overhead, as described in [I-D.ietf-savnet-intra-domain-problem-statement]. To help address these limitations, this document describes an architecture for intra-domain SAV. This architecture focuses on SAV rule generation at external interfaces of routers where intra-domain SAV is applied. SAV at external interfaces facing a neighboring AS, as well as SAV at internal interfaces, is outside the scope of this architecture. This architecture assumes that intra-domain routers are trusted and operate correctly. A compromised intra-domain router is outside the threat model of this architecture.

The architecture is designed to be generally applicable and extensible. It does not specify a particular mechanism, protocol, or algorithm, and does not assume pervasive deployment in an AS. Instead, it provides a basis for describing the conditions under which the architecture can improve SAV accuracy and operational efficiency with respect to existing intra-domain SAV mechanisms. The reader is expected to be familiar with [I-D.ietf-savnet-intra-domain-problem-statement].

# Terminology

{:vspace}
SAV Rule:
: The rule that indicates the validity of a specific source IP address or source IP prefix per router interface. It is used by a router to make SAV decisions.

SAV Agent:
: A logical function that obtains information used for SAV rule generation, generates SAV rules, and provides the generated SAV rules to routers for application at specific external interfaces. A SAV Agent may be implemented on a router or in another system. This document does not require a particular deployment location for the SAV Agent. Depending on the mechanism and the information available, a SAV Agent may use routing information, SAV-specific information, or both.

SAV-specific Information:
: Information specialized for SAV rule generation. SAV-specific information may be provided by routers within an AS, or through operator provisioning (i.e., SAV-specific configurations).

SAV-specific Information Delivery Mechanism:
: A mechanism by which SAV-specific information is made available to a SAV Agent when the information is not locally available to that SAV Agent. Such a mechanism may be realized by reusing an existing mechanism/protocol, extending an existing mechanism/protocol, or defining a new mechanism/protocol.

SAV Information Base:
: A conceptual data store maintained by a SAV Agent. It contains information used for SAV rule generation.

# Architecture

## Overview {#sec-arch-overview}

{{fig-arch}} illustrates the conceptual components and information flow of the intra-domain SAV architecture. The architecture centers on a SAV Agent, which is a logical function that obtains information used for SAV rule generation, generates SAV rules, and provides the generated SAV rules to routers for application at specific external interfaces. A SAV Agent may be implemented on a router or in another system. This document does not require a particular deployment location for the SAV Agent.

A SAV Agent can obtain information used for SAV rule generation from information sources within the AS, including routers within the AS and operator-managed systems. Depending on the mechanism, the information used for SAV rule generation may include routing information, SAV-specific information, or both. Routing information is obtained from the routing system, while SAV-specific information may be provided by routers or through operator provisioning. A SAV Agent processes the available information and generates SAV rules. The generated SAV rules are made available to routers for application at the corresponding external interfaces. Data-plane SAV enforcement is performed by routers at those external interfaces by validating incoming packets against those SAV rules and applying the configured traffic handling policy to packets classified as invalid, as described in [I-D.ietf-savnet-general-sav-capabilities].

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
                   | and/or routing information
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

### Routing Information

Routing information is information maintained by the routing system that represents routing or forwarding state within the AS, such as information in RIBs or FIBs on routers. Such information may be learned through routing protocols or result from routing configuration, but this architecture treats the resulting routing or forwarding state, rather than the operator configuration itself, as routing information for SAV rule generation.

Routing information expresses reachability and forwarding behavior. For example, it can indicate which destination prefixes are reachable through a particular external interface.

An intra-domain SAV mechanism may use routing information as an input for SAV rule generation. For example, routing prefixes associated with an external interface may be used to identify part of the permitted source address space associated with that interface. However, routing information does not always directly or completely represent source-address authorization, especially in asymmetric routing or hidden-prefix scenarios (see [I-D.ietf-savnet-intra-domain-problem-statement]).

An operational advantage of using routing information is that changes in the routing system can be reflected automatically in the routing information available for SAV rule generation. This includes changes in routing state resulting from routing protocol updates or routing configuration changes. Therefore, routing-derived input to SAV rule generation can track routing-visible changes without requiring a separate SAV-specific configuration update for each such change.

When relevant routing information is not locally available to the SAV Agent, it may be obtained through existing routing, management, or control mechanisms. This architecture does not define a new mechanism for delivering routing information.

### SAV-specific Information {#sec-sav-specific-information}

SAV-specific information is information dedicated to SAV rule generation. It may by itself provide sufficient information for SAV rule generation, or may be used together with routing information, depending on the mechanism and the information available.

SAV-specific information may identify the external interfaces connected to the same attached entity and specify the source address space that the entity is authorized to use for sourcing traffic. The interface-to-entity association allows information about the entity to be used when generating SAV rules for the corresponding interfaces, while the authorized source address space can explicitly represent the source addresses permitted for the entity, including information that may not be available from routing information. Such information is useful in scenarios such as asymmetric routing and hidden prefixes.

## SAV-specific Information Delivery {#sec-info-delivery}

A SAV-specific information delivery mechanism makes SAV-specific information available to a SAV Agent when the information is not locally available to that SAV Agent. The information provider and the information receiver need to support a common delivery mechanism. Such a mechanism may be realized by reusing an existing mechanism/protocol, extending an existing mechanism/protocol, or defining a new mechanism/protocol.

A solution that defines a SAV-specific information delivery mechanism should define the information semantics, encoding, transport or management method, update behavior, and error handling appropriate for that mechanism. When SAV-specific information changes, the SAV Agent (or information receiver) needs to be able to obtain the updated information in a timely manner so that the generated SAV rules can remain consistent with the latest state.

The SAV-specific information delivery mechanism should provide appropriate session security protection. This includes authentication of the communicating entities and integrity protection for the delivered information. When SAV-specific information contains private or sensitive operational information, the delivery mechanism also needs to provide confidentiality protection or otherwise prevent disclosure to unauthorized entities.

## SAV Agent and SAV Information Base {#sec-arch-agent}

A SAV Agent is the logical function responsible for obtaining information used for SAV rule generation, maintaining a SAV Information Base, and generating SAV rules.  A SAV Agent may be implemented on a router or in another system.

{{fig-sav-agent}} illustrates the conceptual processing model of a SAV Agent. Depending on the mechanism, the SAV Agent may obtain routing information from the routing system, SAV-specific information from routers or through operator provisioning, or both. The SAV Agent maintains the SAV Information Base and uses the available information to generate SAV rules.

The SAV Information Base is a conceptual data store maintained by a SAV Agent. It contains information used for SAV rule generation, such as routing information associated with external interfaces, SAV-specific information associated with external interfaces or attached entities, and associations between attached entities and external interfaces.

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

A SAV Agent generates SAV rules based on information available in the SAV Information Base. The rule-generation algorithm is mechanism-specific and is not defined by this document.

For an external interface facing an attached entity, this architecture recommends generating allowlist-based SAV rules that identify the source addresses or source prefixes permitted on that interface. When the available information is known to be incomplete, the SAV mechanism needs to account for such incompleteness when generating and applying SAV rules so as to avoid improper blocking of legitimate traffic. Further considerations for incomplete information are described in {{sec-incremental-deployment}}.

When the same entity is attached through multiple external interfaces, the SAV Agent needs to identify the association among these interfaces so that information associated with the entity can be used when generating SAV rules for the corresponding interfaces.

## SAV Rule Installation and Data-plane Enforcement {#sec-data-plane-enforcement}

After SAV rules are generated by a SAV Agent, they are installed on routers that apply intra-domain SAV at the corresponding external interfaces. Data-plane SAV enforcement is the process of validating incoming packets against the installed SAV rules and applying the configured traffic handling policy to packets classified as invalid. 

The action for packets classified as invalid is a matter of local policy and deployment stage.  A deployment can initially use monitoring, logging, sampling, rate-limiting, redirecting, or other conservative actions before enabling strict dropping.  Further considerations for data-plane SAV capabilities are described in {{I-D.ietf-savnet-general-sav-capabilities}}.

# Improvement over Existing SAV Mechanisms

Existing intra-domain SAV mechanisms typically determine the permitted source address space on an external interface using only information locally available at the corresponding router, such as routing information in the local FIB, or rely on explicitly configured source-prefix information. A mechanism based only on local routing information may obtain an incomplete view when the same attached entity is connected through multiple external interfaces or routers, while a configuration-based mechanism can explicitly specify the permitted source address space but requires the corresponding SAV configuration to be maintained and updated by the operator.

The architecture described in this document extends these approaches by allowing a SAV Agent to associate information across the external interfaces connected to the same entity and to use SAV-specific information where needed. The conditions under which these capabilities improve SAV accuracy and operational efficiency are described below.

## Accuracy Improvement

The architecture can improve SAV accuracy when the information available to the SAV Agent more completely represents the source address space permitted on an interface than the information available to existing routing-based SAV mechanisms.

For example, when the same entity is connected through multiple external interfaces or routers, the SAV Agent can use the association among those interfaces to consider routing information associated with the entity more completely, thereby improving accuracy in asymmetric-route scenarios. In addition, SAV-specific information can explicitly identify legitimate source prefixes that are not represented in routing information, such as source-only prefixes in hidden-prefix scenarios.

When the required information is sufficiently complete, correct, and timely, the SAV Agent can more accurately determine the source address space permitted on an interface, thereby reducing SAV classification errors. In particular, the additional information described above can avoid improper blocking of legitimate traffic in asymmetric-route and hidden-prefix scenarios. This improvement does not require pervasive deployment and can be achieved at the external interfaces where the required information is available.

## Operational Efficiency Improvement

The architecture can improve operational efficiency by using routing information for source prefixes that are visible in the routing system. Because routing information changes together with routing or forwarding state, routing-derived source-prefix information can track such changes automatically without requiring a separate SAV-specific configuration update for every routing change.

When routing information is used for routing-visible source prefixes, this can reduce the amount of independently maintained SAV-specific configuration and the synchronization overhead between routing and SAV configuration. SAV-specific provisioning remains available for explicit source-address authorization, including source-only prefixes and other information that is not represented in routing information. Compared with purely configuration-based SAV mechanisms, this can reduce manual configuration overhead while retaining the ability to represent source-address authorization that is not visible in routing information.

# Operational Considerations

## Incremental Deployment {#sec-incremental-deployment}

Incremental deployment can occur in two different aspects: the incremental availability of information required for SAV rule generation, and the incremental application of SAV rules at routers.

The information required by a mechanism may be available for some external interfaces or attached entities, but unavailable or incomplete for others. For example, a mechanism that relies on SAV-specific information may have such information available only for a subset of attached entities. In such cases, a SAV Agent may be able to determine the permitted source address space for some external interfaces or attached entities, but may be unable to do so with sufficient completeness for others. When the information required by the mechanism is unavailable or incomplete, operators are encouraged to use conservative traffic handling policies. Such policies may include logging, monitoring, rate-limiting, or other non-dropping actions before strict blocking is enabled.

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

