---
title: Leveraging the cloud (future)
last_updated: November 6, 2017
sidebar: main_sidebar
permalink: introduction_leveraging-the-cloud.html
summary:
---

{% include note.html content="The environment described below represents future work that is outside the scope of this document. We describe it here to record our thinking on the topic, but we have no plans to implement a hybrid environment at the present time." %}

The on-premises single sign-on environment described in the previous section, even though designed for high availability, may still be vulnerable to large-scale adverse events such as natural disasters (Hurricane Sandy, 2012), public infrastructure failures (Northeast power outage, 2003), or Tier-1 Internet provider outages (Level3 Communications, 2013). If such an event were to occur and "take out" both New School data centers or both New School Internet connections, the impact would be felt across more than just the other New School applications hosted in our on-premises data centers. Cloud-hosted applications, such as Google G Suite, Workday, and Canvas would also become unavailable, because they depend on the New School single sign-on service to enable users to log in.

Figure 2 shows how Amazon Web Services (AWS) could be used to add a cloud-based component to the on-premises deployment described in the previous section. In this hybrid design, on-campus users would continue to use the on-premises server environment to authenticate, providing better performance, while off-campus users would use the cloud-based environment.

{% include image.html file="about/fig02-cloud-env.png" alt="Cloud Environment Diagram" caption="Figure 2. Hybrid on-premises/cloud CAS 5 environment" %}

To keep on-campus users authenticating against the on-premises server environment while directing off-campus users to the hybrid environment, the New School domain name servers would be configured such that the internal servers (used by on-campus devices) and external servers (used by off-campus devices) return different results when resolving the user- (or client-) facing domain name for the single sign-on service. The internal name servers would continue to resolve the domain name to a virtual address on the F5 load balancers, resulting in on-campus users being directed to the on-premises environment as described previously. The external name servers, however, would resolve the domain name to a virtual address on an AWS Elastic Load Balancer.

The user-facing Elastic Load Balancer (at the top of the diagram) could be a single instance or, to ensure high availability, multiple instances (perhaps even across multiple availability zones). As shown in the diagram, the load balancer would route connections to a mixed pool of on-premises and cloud-based CAS servers. Utilizing the on-premises servers reduces the number of cloud-based servers needed, although there must still be enough cloud-based servers to ensure availability and response time when the on-premises environment is unavailable. The cloud-based CAS servers would be configured to be members of the same ticket storage and service registry replication pools as the on-premises servers, to ensure a seamless user experience regardless of which servers are accessed.

The cloud-based environment would also contain cloud-based instances of Active Directory and Luminis LDAP. Additional Elastic Load Balancers would be used to route directory queries to mixed pools of on-premises and cloud-based directory servers. As above, utilizing the on-premises servers reduces the number of cloud-based servers (and the amount of storage) needed, although there must still be enough cloud-based servers of each type to ensure availability and response time when the on-premises environment is unavailable. The cloud-based servers would be configured to replicate with their on-premises counterparts, ensuring a seamless user experience regardless of which servers are accessed.

The addition of a cloud-based component to the on-premises deployment described in the previous section will ensure that the New School single sign-on service is resilient even in the face of large-scale adverse events that "take out" the on-premises environment. The degree of resilience provided by the cloud environment can be increased or decreased through the deployment of additional server instances, the use of multiple availability zones, or even the use of multiple cloud providers.
