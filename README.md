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
    
## authentication in a Kubernetes:
Different users that interact with kubernetes cluster :
- Administrators that access the cluster to perform administrative tasks  
- The developers that access to cluster to test or deploy applications
- End users who access the applications deployed on the cluster 
- third party applications accessing the cluster for integration purposes.

**How to secure management access to the cluster through authentication and authorization**
we are left with two types of users:
- **Users** Humans, such as the Administrators and Developers.
- **Service Accounts** Robots such as other processes/services or applications that requires access to the cluster.
  
- Kubernetes doesn't manage users in cluster So it can't create users or view the list of users it relies on an external source like:
  **file with user details** or **certificates** or a third party identity service like **LDAP** to manage these users.
  However in case of Service Accounts kubernetes can manage them, You can create and manage service accounts using the Kubernetes API.
  #kubernetes create serviceaccount sa1
  
- All user access is managed by the **API server** by authenticates all of these requests.
- how does the kube-api server authenticate?
      - usernames and static password file
      - static token file for each user 
      - certificates
      - LDAP  
  **For usernames and static password file** You can create a list of users and their passwords in a csv file,
      The file has three columns (password123,user-name,User-ID) then pass the file name as an option to the kube-api server file.
      /etc/kubernetes/manifests/kube-apiserver.yaml (--basic-auth-file=users.svc)
      **SO when you accessing the API server specify the user and password like this:** 
        #curl -v -k https://master-node-ip:6443/api/v1/pods -u "username:password123"   
      - you can assign users to specific groups (password123,user-name,User-ID,group1).
  
  **For static token file** you can pass the token file as an option token-auth-file to the kube-api server (--token-auth-file=users.svc)
       While authenticating specify the token as an Authorization bearer token to your requests like this.
        #curl -v -k https://master-node-ip:6443/api/v1/pods --header "Authorization: Bearer lwesniwoniwnwiwenin"  

## Service accounts:
  service accounts are used by machines, It could be an account used by an application to interact with the Kubernetes cluster.
  When the service account is created,you should also creates a token That used by the external application while authenticating to the 
  Kubernetes API that have expiry date #k create token serviceaccount-name,then creates a secret object and stores that token inside the secret object.  
  This token can then be used as an authentication bearer token while making Request call to the Kubernetes API. 
    - EX: monitoring application like **Prometheus** uses a service account to pull the **Kubernetes API** for performance metrics.
    - **Jenkins** uses service accounts to deploy applications on the Kubernetes cluster. 
  
 There is a default service account For every namespace in Kubernetes by default, 
 Whenever a pod is created, the default service account and its token are automatically mounted to that pod as a volume mount to location inside pod 
 that help to access kubernetes api. 
 Now, remember that the default service account is very much restricted.It only has permission to run basic Kubernetes API queries, SO you can create 
 a new service account and modify a pod to include a service account field **serviceAccountName: name-of-service-account** 
 
  
## SSL TLS certificates:
  A certificate is used to guarantee trust between two parties during a transaction. 
  For example, when a user tries to access a web server,**TLS certificates** ensure that the communication between the user and the server is encrypted
  Encryption and Decryption is done by **Asymmetric Encryption** that uses a pair of keys,**Private key, and a public key**

  **Ex:** We generate a public and private key pair on the server by running **openssl** command, Server will send the **puplic key** to the user,
          the user has symmetric key (user private key that used for encryption or deryption). 
          When the user first want to accesses the webserver using HTTPS (https://mybank.com),then the user gets a copy of the server public key 
          then the user send encrypted symmetric key with the public key provided by the server(user private key + server puplic key).
          then The user sends this to the server (user private key + server puplic key).The server uses its private key to decrypt the message that 
          sent by user (user symmetric/private key + server puplic key + server private key),and retrieve the symmetric key(user private key) from it. 
  
          **TLS certificates** It has information about who the certificate is issued to,the public key of that server,the location of that server.
          We want to secure more the communication between user and server by using "TLS certificates with your key"
          to ensure that you use a legitimate key: 
          SO, When the server sends the key, it does not send the key alone, it sends a certificate that has the key in it.  
          But, How do you look at a certificate and verify if it is legit? and who signed and issued the certificate?
          If you generated a certificate then you will have to sign it by yourself **"self-signed certificate"** But it is not a safe certificate,
          because you have signed it. So we need trusted way to sign our certificate. 
          **your browser** does that for you,All of the web browsers are built-in with a certificate validation mechanism wherein the browser checks
          the certificate received from the server and validates it to make sure it is legitimate. 
          Then how do you create a legitimate certificate for your web servers that the web browsers will trust? That's done by 
          **certificate authorities or CAs** They are well-known organizations that can sign and validate your certificates for you 
          Ex: (Symantec,DigiCert,Comodo,) The way this works is:
           - you generate a certificate signing request or CSR using the key you generated and the domain name of your website using **openssl command**
           - The Certificate Authorities verify your information.
           - Once it checked it sign the certificate and send it back to you.(You now have a certificate signed by a CA that the browsers trust.)
          How do the browsers know that the CA itself was legitimate?
          How would the browser know Symantec is a valid CA and that the certificate was in fact signed by Symantec ?
          - The CAs themselves have a set of public and private key pairs.
          - The CAs use their private keys to sign the certificates.
          - The public keys of all the CAs are built-in to the browsers.
          - The browser uses the public key of the CA to validate that the certificate was actually signed by the CA themselves.
  
          Summary:
          - We use asymmetric encryption to encrypt our messages with a pair of public and private keys.
          - An admin have a pair of keys (user public and private keys) to secure SSH connectivity to the servers, by sending puplic key to server.
          - The server have a pair of keys (server public and private keys) to secure a HTTPS traffic,But for this it needs firstly to sends CSR.
          - The server sends a (certificate signing request+server puplic key) to a CA like Symantec,the CA uses its private key to sign the CSR.
                  **Remember** all users have a copy of the CAs public key
          - The signed certificate is then sent back to the server,the server configures the web application with the signed certificate.
          - Whenever a user accesses the web application (https:/my-bank.com)the server first sends the ( signed certificate + server public key) to user.
          - As the user have already (CA puplic key) so,
          - The user's browser reads the certificate and uses the CAs public key to validate and retrieve the server's public key.
          - the user generates a symmetric key (user private key) that it wishes to use going forward for all communication, 
          - The symmetric key is encrypted using the servers public key and sent back to the server.
          - The server uses its private key to decrypt the message, and retrieve the symmetric key,
          The symmetric key is used for communication going forward, 
        
          SO, the administrator generates a key pair for securing SSH, 
          - the web server generates a key pair for securing the website with HTTPS
          - the Certificate Authority generates its own set of key pair to sign certificates.
          - The end-user, though, only generates a single symmetric key.
  
  ## Securing your Kubernetes cluster with TLS Certificates:
      Thers're three types of certificates:
      - server certificates configured on the servers
      - root certificate configured on the CA servers
      - client certificates configured on the clients
  
     The two primary requirements:
     - all the various **services** within the cluster to use **server certificates**
     - all **clients** to use **client certificates**
    
    Different components within the kubernetes cluster and identify the various servers and clients:
    ## Server Certificate for Servers:
    - **API server** exposes an HTTPs service that other components as well as external users use to manage the Kubernetes cluster.
        (So it is a server and it requires certificates to secure all communication with its clients,So we generate a certificate and key Pair.)
          **"apiserver.crt and apisever.key"**
    - ** ETCD server** stores all information about the cluster so it requires a pair of certificate and key for itself.
          **"ETCD server.CRT and ETCD server.key"**
    - **KUBELET server** expose an HTTPs API in point that the Kube-api server talks to to interact with the worker nodes 
         that requires a certificate in keypair **"kubelet.crt and kublet.key"**
  
   ## Client Certificate for Clients:
   - **admin user** client that access the kubeapi server are the administrators through Kubectl requires a certificate and key pair 
       to authenticate the kube-api server **"admin.crt and admin.key"**
   - **Scheduler** client that talks to the kube-api server to look for pods that require scheduling, So it also requires a certificate for authentication
       to the kube-api server. **"scheduler.crt and scheduler.key"**
   - **kube-controller manager** client that needs to accesses the kube-api server So it also requires a certificate for authentication to
       the kube-api server **"controller-manager.crt and controller-manager.key"**
   - **kube-proxy** client that requires a certificate for authentication to the kube-api server.**"kube-proxy.crt and kube-proxy.key"**
   
   - **kube-api server with ETCD** the kube-api server is a client and the only server that talks to the ETCD server,So it needs to authenticate.
            The kubeapi server can use the same keys that it used earlier **Or** you can generate a new pair of certificates specifically
             for the kube-api server to authenticate to the etcd server **"apiserver-etcd-client.crt and apisever-etcd-client.key"** 
  
   - **kube-api server with Kubelet** The kube-api server also talks to the kubelet server on each of the individual nodes for monitoring the nodes.
        **"apiserver-kubelet-client.crt and apisever-kubelet-client.key"** 
  
   **We need a certificate authority to sign all of these certificates, Kubernetes requires you to have certificate authority for your cluster.**
      and it has its own pair of certificate **"ca.crt and ca.key"** also you can have more than one certificate authority:
      - one for all the components in the cluster
      - another one specifically for ETCD: In that case the ETCD servers certificates and the ETCD servers client certificates, 
          which in this case is the api-server client certificate will be all signed by the ETCD server CA.
  
  ## TLS in Kubernetes:
    **Create CA private key and root certificate file**
     use Open SSL tools to generate the certificates, 
     1-First we create a private key using the openssl command:
     - #openssl genrsa –out ca.key      ....>> ca.key created
     2-generate a certificate signing request with all of your details:
     - #openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr      ....>> ca.csr created 
     3-sign the certificate using the openssl x509 command:
     - #openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
  
    **Another Examble for Admin User**
     use Open SSL tools to generate the certificates, 
     1-First we create a private key using the openssl command:
     - #openssl genrsa –out admin.key  2048     ....>> admin.key created
     2-generate a certificate signing request with all of your details:
     - #openssl req -new -key admin.key -subj "/CN=kube-admin" -out admin.csr      ....>> admin.csr created 
     3-sign the certificate using the openssl x509 command, But this time, you specify the CA certificate and the CA key to sign your certificate : 
     - #openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
  
    **How do you differentiate this admin user from any other users?**
      The user account needs to be identified as an admin user and not just another basic user,
      You do that by adding the group details for the user in the certificate.
      In this case a group named SYSTEM:MASTERS exists on kubernetes with administrative privileges while generating a CSR like that:
      - #openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr
  
    **Now what do you do with these certificates.**
      For Examble if we used client certificate instead of a user and password in a REST API call you make to the kube-api server.
      - You specify the key the certificate and the ca certificate as options
      curl https://kube-server:6443/api/v1/pods  --key admin.key  --cert  admin.crt  --cacert  ca.crt  
      - the other way is to move all of these parameters into a configuration file called **kube-config**
        Within that specify the **API server** endpoint details the certificates to use etc..
  
    **Remember**
    - whenever you configure a server or a client with certificates you will need to specify the **CA root certificate**
    
  ** For Server Certificats for Servers: ETCD server** 
    - We follow the same procedure as before to generate a certificate for ETCD , We will name it ETCD-SERVER 
      ETCD Server can be deployed as a cluster across multiple servers as in high availability environment:
      In that case to secure communication between the different members in the cluster, we must generate additional **peer certificates** 
  
    **API server**
    - Api Server have many names and aliases within the cluster and all of these names must be present in the certificate generated for 
      the kube-api server: kube-api server , kubernetes , kubernetes.default , kubernete.default.svc 
        kubernetes.default.svc.cluster.local , The IP address of the host running the kube-api server  
    - So we use the same set of commands as earlier to generate a key "openssl genrsa"
    - In the certificate signing request you specify the name KUBE-APIserver "openssl req " , But how do you specify all the alternate names ? 
          - you must create an openssl config file, Create an **openssl.cnf** file and specify the alternate names in the alt name section of the file
            that Include all the DNS names the API server goes by as well as the IP address.
          - Pass this config file as an option while generating the certificate signing request.
              #openssl req -new -key apiserver.key -subj "/CN=kube-apiserver" -out apiserver.csr  -config openssl.cnf 
  
     - Finally, sign the certificate using the CA certificate and key. You then have the kube api server certificate.
      #openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out apiserver.crt  -extensions v3_req -extfile openssl.cnf -days 1000
  
    **where we are going to specify these keys?**
    - Remember to consider the apiserver client certificates that are used by the api server while communicating as a client to the etcd and kubelet servr
    - The location of these certificates are passed in to the kube-api servers :
      1- First the CA file needs to be passed in "every component needs the CA certificate to verify its clients."**--client-ca-file=**
      2- Then the apiserver certificates under the tls-cert options **--tls-cert-file=**,**--tls-private-key-file=**
      3- then specify the client certificates used by the kube-api server to connect to the etcd server 
        **etcd-cafile=**,**etcd-certfile=**,**etcd-keyfile=**
      4- the kube-api server client certificate to connect to the kubelets 
        **kubelet-certificate-authority=**,**kubelet-client-certificate=**,**kubelet-client-key=**
  
  **Kubelet server**
    The kubelet server is an HTTPS API server that runs on each node, responsible for managing the node.
    That's where the API server talks to to monitor the node as well as any information regarding what pods to schedule on this node.
    So, you need a key-certificate pair for each node in the cluster, and they will be named after their nodes. Node01, node02 and node03
    Once the certificates are created, use them in the kubelet-config file:
    - And as always you need to specify the root CA certificate **clientCAFile**, 
    - And then provide the kubelet node certificates You must do this for each node in the cluster **tlsCertFile**, **tlsPrivateKeyFile**
  
    **The API server needs to know which node is authenticated and give it the right set of permissions**
      So it requires the nodes to have the right names in the right format, Since the nodes are system components,
        the format starts with the system Followed by node then the node name **system:node:node01** 
    **How would the api server give it the right set of permissions?**
        the nodes must be added to a group named SYSTEM:NODES 
  
  ## view certificates in an existing cluster
    If you're asked to perform a health check of all the certificates in the entire cluster:
    - First of all, it's important to know how the cluster was set up as they use different methods to generate and manage certificates:
        If you were to deploy a Kubernetes cluster from scratch,then you generate all certificates by yourself.
        If you were to deploy a Kubernetes cluster using kubeadm, then it's automatically generating and configuring the cluster for you.
      **For environment set up by Kubeadm**
         to perform a health check, start by identifying all the certificates used in the system,So The idea is to create a list of:
         Types of certificate files used, certificate paths, the names configured on them, the alternate names configured if any,the organization ,
         the certificate account belongs to, the issuer of the certificate,  the expiration date on the certificate.
         **Start with a certificate files used** 
           - look for the kube-apiserver definition file "/etc/kubernetes/manifests/kube-apiserver.yaml"
           - Next, take each certificate and look inside it to find more details about that certificate, For example, we will start with:
              the API server certificate file **--tls-cert-file=/etc/kubernetes/pki/apiserver.crt**
          - Run the openssl x509 command and provide the certificate file as input to decode the certificate and view details.
              #openssl x509 -in /etc/kubernetes/pki/apiserver.crt   -text  -noout
          - Start with a name on the certificate under the subject section **Subject CN=kube-apiserver**
          - then the alternate names The kube-apiserver has many,so you must ensure all of them are there
          - check the validity section of the certificate to identify the expiry date 
          - then the issuer of the certificate , This should be the CA who issued the certificate, Kubeadm named the Kubernetes CA as Kubernetes itself.
  
    **When you run into issues,you want to start looking at logs**
      - If you set up the cluster from scratch by yourself, and the services are configured as native services in the OS, 
          #journalctl -u etcd.service -l
      - In case you set up the cluster with Kubeadm, then the various components are deployed as pods.
          #kubectl logs etcd-master 
      - **Sometimes,** if the core components Kubernetes API server or the etcd server are down, 
        the Kube control commands won't function, In that case, you have to go one level down to Docker to fetch the logs 
         - List all the containers using the #docker ps -a 
         - Then view the logs using the #docker logs container-id
      
  
  ## Tls Certificate:
      What is the CA server and where is it located in kubernetes?
      The CA is really just the pair of key and certificate files on master node ,Whoever gains access to these pair of files can sign any certificate
       for the Kubernetes,They can create as many users as they want and assign to have access to ckuster with different previliages for
       these users,These files need to be protected So we place them on a server called "CA server" on master node.
  
      How to automate managing the certificates sign request as well as to rotate certificates when they expire?
      Kubernetes has a built-in Certificates API.
      - Adminstrator Creates CertificateSigningRequrst Object .
      - Review the Requests , Then Approve it by the adminstrator of the cliuster.
      - Share the certificates with users  
  
      EXamble:
          - A user first creates a key  >>  #openssl genrsa -out  jane.key 2048
          - generates a CSR using the key   >>    #openssl req -new -key  jane.key  "/CN=jane"  -out jane.csr   >> sends the request to the administrator.
          -  The administrator takes the key and creates CSR Object "that looks like any object inside k8s pod,service,.."
             The kind, is CertificateSigningRequest & the spec section, groups: specify the groups the user should be part of , usages: list the usages 
                of the account, The request: field is where you specify the certificate signing request sent by the user it must be encoded.
                          #cat  jane.csr | base64
          - Once the Object is done then all CSR can viewed by the adminstrator of the cliuster. >> # kubectl get csr 
          -  Review the Requests , Then Approve it    >> #kubectl  certificate  approve  jane 
          - Kubernetes assigns the certificate using the CA key pairs and generates a certificate that can be shared with  the user.
          - to view the certificate     >> #kubectl get csr jane -o yaml   
          - The generated certificate but it is in a base64 encoded format So to decode it to be shared with user >># echo "KDI...KKSNW" | base64 --decode
            
      Controller Manager:Is responsible for all the certificate related operations that have different controller for cert like"CSR Approving,CSR signing"
          - if anyone has to sign certificates, they need the CA servers, roote certificate, and private key, that existed inside 
            "kube-controller-manager.yaml"    >> # cat /etc/kubernetes/manifests/kube-controller-manager.yaml  >> "--cluster-signing-cert,key-file" 
  
  
  ## KubeConfig in Kubernetes:
      Instead of each time the user uses the certificate file and key to query the kubernetes Rest API for a list of pods using Curl or kubectl command:
        - curl https://my-kube-playground:6443/api/v1/pods    --key admin.key   --cert  admin.crt   --cacrt   ca.crt
        - kubectl get pods --server my-kube-playground:6443  --client-key admin.key  --client-certificate  admin.crt  --certificate-authority  ca.crt
|        then validated by the API server to authenticate the user,is a tedious task SO move these information to a configuration file called as 
          "KubeConfig" and specify it as optionin your command. >> #kubectl get pods --kube-config  config 
        - By default the kubectl tool looks for a file named "config" under a directory .kube under the users home directory, So So if you create the 
          KubeConfig file there, you don’t have to specify the path to the file explicitly in the kubectl command. >> #kubectl get pods 
  
        -  The config file has 3 sections. Clusters, Users and Contexts:
                  Clusters: are the various kubernetes clusters that you need access to. ex:"devlopment"
                  users: user accounts that needs access to these clusters , each user have different priviliages   ex: "developer"
                  contexts: connect these users with its cluster  ex: "devlopment@developer"
        - That way you don’t have to specify the user certificates and server address in each and every kubectl So the:
               The server specification " --server my-kube-playground:6443 "  goes to "cluster sec"
                 the admin users keys and certificates " --client-key admin.key  --client-certificate  admin.crt  --certificate-authority  ca.crt" 
                  goes into the users section.
  
       **The kubeConfig is in a YAML**  
            - It has apiVersion: set to v1 , The kind: is config.... And then it has 3 sections:
              1- clusters       2- users      3- contexts
            - how does kubectl know which context to chose from if you have multiple contexts?
                 the default context to use by adding a field "current-context" to the kubeconfig file.
            - to view the current kubeConfig file :
                  #kubectl config view 
            - to view the custom kubeConfig file:
                   #kubectl config view --kubeconfig=my-custom-config
            - to change the current-context to the prod-user@production context
                  #kubectl config use-context  prod-user@production
            - certificates in kubeConfig:
                  You have seen path to certificate files mentioned in kubeconfig ex: " certificate-authority: /etc/kubernetes/pki/a.crt "
                  But there's another way to specify the certificate credentials by passing the encoded certificate like this : 
                          ex: " certificate-authority-data: KFIENFI.....OIENV  "
                    
    
    ## API Groups in kubernetes:
       what is the Kubernetes API?
          - Any interacting with the cluster is done through "the Kube API server" by using **kubectl command** or **curl**
            ex:  to check the versio      >> curl https://kube-master:6443/version 
                  to get the list of pods  >>  curl https://kube-master:6443/api/v1/pods
          
          - What these API Path (/version & /api ) ?
               The Kubernetes API is grouped into multiple such groups based on their purpose ex: "/apis, /version, /healthz, /api, /metrics, /logs"
                /version : is for viewing the version of the cluster 
                /healthz &  /metrics : to monitor the health of the cluster.
                /logs : Is for integrating with third party logging applications.
  
          - The DIferrence between these Apis (/api  &&  /apis) ?
              These APIs are categorized into two: 
                 - The core group **/api**: is where all core functionality exists. Such as namespaces, pods, replicas,.....etc ex: /api/v1/pods,nodes,,,,
                 - The named group **/apis**: all the newer features are available to these named groups...Such as groups for
                 /apps, /storage.k8s.io, /networking.k8s.io ....and inside each one you will have multiple resources that associated with multiple actions  
                like for **/apps** group under it deployments, replicasets , statfulsets you can do action list,delete,get >>ex: /apis/apps/v1/deployments  
                     
           - To know what the API groups on your Kubernetes cluster ?   you have 2 options   
                # curl  http://localhost:6443  -k  --key admin.key   --cert  admin.crt   --cacrt   ca.crt
              OR))
                Start a kubectl proxy client that launches a proxy service locally on port 8001 and uses credentials and certificates from your kubeconfig
                 file to access the cluster.        >>        #kubectl  proxy       then  >> #curl  http://localhost:8001  -k
        
          - NOTE:
              The Kube proxy and kubectl proxy well they are not the same
              kubectl proxy: It is used to enable connectivity between parts and services across different nodes in the cluster.
                
  
  
  
  
  
