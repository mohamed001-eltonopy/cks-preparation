# cks-preparation

## Understanding of the Kubernetes attack surface 

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

**what CIS benchmarks** the Center for Internet Security whose mission is to make the connected world a safer place by developing, validating, 
and promoting timely best practice solutions that help people, businesses, and governments protect themselves against pervasive cyber threats .
The CIS website provides cybersecurity benchmarks for over 25 different vendors like 
-OS: Linux, Windows
-Public cloud platforms: Google, Azure, AWS, and others.
-Server: K ubernetes, Docker,...

**CIS also provide necessary tools that can help run assessments and remediate them like CIS-CAT tool.
**The CIS-CAT tool** help in the automated assessment of the server, It compares the current configuration on the server against the recommended security 
                best practices in the benchmark document and reports issues if any are found in html format.
**test**
      We have installed the **CIS-CAT Pro Assessor** tool called Assessor-CLI, under /root.
      Please run the assessment with the Assessor-CLI.sh script inside Assessor-CLI directory and **generate a report** called index.html in the output 
      directory /var/www/html/.Once done, the report can be viewed using the Assessment Report tab located above the terminal.
      **Run the test in interactive** mode and use below settings:
      Benchmarks/Data-Stream Collections: : CIS Ubuntu Linux 20.04 LTS Benchmark v1.1.0
      Profile : Level 1 - Server
      **sol**          
      Use sh ./Assessor-CLI.sh to see all available options. Use the correct option to set the output directory which is (-rd) and override the default 
      report name and Run the test in interactive using option (-i).
      -i,--interactive                                  Proceed interactively
      -rd,--reports-dir <REPORTS-DIR>                   Path to a directory specifying the location to which output reports are saved
      -rp,--report-prefix <REPORT-PREFIX>               Override the default report name.
      -nts,--no-timestamp                               Do not include the auto-generated timestamp as part of the report name.
                                                   

      cd /root/Assessor-CLI
      sh ./Assessor-CLI.sh -i -rd /var/www/html/ -rp index -nts 
    
    
## Kube-bench tool:
  is an open-source tool from Aqua security that can perform automated assessments to check whether Kubernetes is deployed as per security best practices.
  by running the checks documented in the [CIS Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes)
  **test**
  Install kube bench in /root directory
  version : v0.4.0
  ---
  #curl -L https://github.com/aquasecurity/kube-bench/releases/download/v0.4.0/kube-bench_0.4.0_linux_amd64.tar.gz -o kube-bench_0.4.0_linux_amd64.tar.gz
  #tar -xvf kube-bench_0.4.0_linux_amd64.tar.gz
  **Run the kube-bench test**
  #./kube-bench --config-dir `pwd`/cfg --config `pwd`/cfg/config.yaml

  
## security primitives in Kubernetes    
  Let's begin with the host that formed the cluster itself,all access to these hosts must be secured:
  -  root access disabled
  - password based authentication disabled
  - only SSH key based authentication to be made available.

  The first line of defense is to secure **Kube-ApiServer** Controlling access to the API server itself, by using authentication and authorizations
  that define who can access the cluster "authentication" and what can they do "authorizations" ?  
  **For authentication**
  Who can access the API server? is defined by the Authentication mechanisms,There are different ways that you can authenticate to the API server:
      - usernames and passwords
      - usernames and tokens 
      - certificates
      - LDAP  
      - serviceaccounts  
  **For authorization**
  what can they do  is defined by authorization mechanisms, There are different ways that you can authorize to the API server:
  - Role Based Access Control " where users are associated to groups with specific permissions "
  - Attribute based access control
  - Node Authorizers
  - webhooks
 
  **All communication with the cluster, between the various components such as the ETCD cluster, kube controller manager, scheduler, api server, 
   as well as those running on the worker nodes such as the kubelet and and kubeproxy is secured using TLS Encryption**
    
  **What about communication between applications within the cluster**
  By default all PODs can access all other PODs within the cluster,You can restrict access between them using **Network Policies**. 
    
    
    
                
                
