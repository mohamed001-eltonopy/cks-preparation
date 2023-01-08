# cks-preparation

## understanding of the Kubernetes attack surface 

**the fources of Cloud-Native Security**
There were multiple areas that were vulnerable to attack,
The infrastructure that hosted the Kubernetes cluster was not properly secured and enabled access to ports on the cluster from anywhere.

- **Cloud Security,** It refers to the security of the entire infrastructure hosting the servers.(DataCenter, Network, Servers)
  The infrastructure that hosted the Kubernetes cluster was not properly secured and enabled access to ports on the cluster from anywhere,
  If network firewalls were in place, we could have prevented remote access from the attacker's system.

- **Cluster security,** The attacker was easily able to gain access through the Docker daemon exposed publicly as well as the Kubernetes dashboard 
  (authentication, authorization, NetworkPolicy,Admissions) that was exposed publicly without proper authentication or authorization mechanisms.
  This could have been prevented if security best practices were followed in securing the Docker daemon, the Kubernetes API,as well as any GUI 
  we used to manage the cluster such as the Kubernetes dashboard.

-  **Container security,** The attacker was able to run a container in privileged mode,install whatever application without any restriction.
  (Restrict Images, Supply Chain, Privileged, SandBoxing).
  These could have been prevented if restrictions were put in place to only run images from a secure internal repository and if running containers
  in privileged mode was disallowed, also containers could be isolated in a better way using SandBoxing. 

-  **The application code security,** Hard coding applications with database credentials or passing critical information through environment variables

- **Securing critical information with secrets and vaults**

-  **Enabling mTLS encryption to secure pod-to-pod communication.**


## Cluster Setup And Hardening 

