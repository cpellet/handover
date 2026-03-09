---
status: Interrupted
technical_owner: Fawad Hussain Syed
strategic_owner: Fawad Hussain Syed
repository: N/A
stack: Kubernetes, Rackspace Spot, Docker
---
**Background:** Platforms such as [[EPIC Development|EPIC]], the [[JIAF Dashboards]] and [[RAG Development|OCHA RAG]] require a server to run on, enabling colleagues to use them without running code on their computers. Due to budget restrictions, access to [AWS](https://aws.amazon.com/) and [Azure](https://azure.microsoft.com/en-us) was heavily restricted, and fully prohibited for new projects. Low-cost and self-managed alternatives enable us to deploy our work with project-earmarked funding.
### Kubernetes
A [Kubernetes](https://kubernetes.io/) cluster running on [Rackspace Spot](https://spot.rackspace.com/) infrastructure provided us with full autonomy over the deployment of multiple platforms over multiple month, with a maximum monthly spending of 17.65 USD. An equivalent deployment on UN platforms (Azure) was estimated to cost upwards of 760 USD (43x more expensive).

Operating a Kubernetes cluster however requires specialized expertise, and is often facilitated by costly [consulting services](https://aurotekcorp.com/top-10-kubernetes-consulting-services-in-2025/). Unless OCHA invests in this technology, and recruits specialized talent to maintain such services in-house, maintaining the HPCS Cluster is not foreseeably viable.

%% More details on the HPCS cluster deployment are available upon request, though the deployment code cannot be open-sourced for prior IP reasons %%