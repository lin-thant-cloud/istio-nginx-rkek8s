https://rke.docs.rancher.com/

# copy keypair 
 scp -i k8s-test.pem k8s-test.pem ubuntu@13.215.250.136:/home/ubuntu/

 # verify local connection 
 ip a
 ssh -i k8s-test.pem ubuntu@local ip for all nodes


# docker install on 2 nodes
curl https://releases.rancher.com/install-docker/20.10.sh | sh


sudo cat /etc/os-release

https://docs.docker.com/engine/install/ubuntu/


# Add user to docker group
sudo usermod -aG docker ubuntu

verify docker ps

Download RKE binary
https://github.com/rancher/rke/#latest-release
https://github.com/rancher/rke/releases/
wget https://github.com/rancher/rke/releases/download/v1.6.2/rke_linux-amd64

verify 
ubuntu@k8s-node1:~$ ls -la
total 60312
drwxr-x--- 4 ubuntu ubuntu     4096 Oct 19 12:48 .
drwxr-xr-x 3 root   root       4096 Oct 19 11:47 ..
-rw------- 1 ubuntu ubuntu     1256 Oct 19 12:37 .bash_history
-rw-r--r-- 1 ubuntu ubuntu      220 Mar 31  2024 .bash_logout
-rw-r--r-- 1 ubuntu ubuntu     3771 Mar 31  2024 .bashrc
drwx------ 2 ubuntu ubuntu     4096 Oct 19 11:58 .cache
-rw-r--r-- 1 ubuntu ubuntu      807 Mar 31  2024 .profile
drwx------ 2 ubuntu ubuntu     4096 Oct 19 12:42 .ssh
-rw-r--r-- 1 ubuntu ubuntu        0 Oct 19 11:59 .sudo_as_admin_successful
-rw-rw-r-- 1 ubuntu ubuntu      165 Oct 19 12:48 .wget-hsts
-rw-r--r-- 1 root   root        391 Oct 19 12:44 cluster.yaml
-r-------- 1 ubuntu ubuntu     1674 Oct 19 12:38 k8s-test.pem
-rw-rw-r-- 1 ubuntu ubuntu 61711319 Sep 17 23:32 rke_linux-amd64

mv rke_linux-amd64 rke
chmod +x rke
sudo mv rke /usr/local/bin/

rke version 
INFO[0000] Running RKE version: v1.6.2                  
FATA[0000] [state] Failed to create Kubernetes Client: stat ./kube_config_cluster.yml: no such file or directory 


creat cluster.yaml
nodes:
  - address: 
    user: centos
    role: [controlplane, worker, etcd]
    hostname_override: master01 
  - address: 
    user: centos
    role: [controlplane, worker, etcd]
    hostname_override: master02 
  - address: 
    user: centos
    role: [controlplane, worker, etcd]
    hostname_override: master03 
services:
  etcd:
    snapshot: true
    creation: 12h
    retention: 168h

# modiify cluster.yaml
# Cluster level SSH private key
# Used if no ssh information is set for the node
ssh_key_path: /home/ubuntu/k8s-test.pem
nodes:
  - address: 10.0.1.217
    user: ubuntu
    role: [controlplane, worker, etcd]
    hostname_override: k8s-node1
  - address: 10.0.1.205
    user: ubuntu
    role: [controlplane, worker, etcd]
    hostname_override: k8s-node3
  - address: 10.0.1.12
    user: ubuntu
    role: [controlplane, worker, etcd]
    hostname_override: k8s-node2
services:
  etcd:
    snapshot: true
    creation: 12h
    retention: 168h
~                             

https://rke.docs.rancher.com/example-yamls


# Run k8s cluster
rke up --config cluster.yaml

output
ubuntu@k8s-node1:~$ rke up --config cluster.yaml
INFO[0000] Running RKE version: v1.6.2                  
INFO[0000] Initiating Kubernetes cluster                
INFO[0000] [certificates] GenerateServingCertificate is disabled, checking if there are unused kubelet certificates 
INFO[0000] [certificates] Generating admin certificates and kubeconfig 
INFO[0000] Successfully Deployed state file at [./cluster.rkestate] 
INFO[0000] Building Kubernetes cluster                  
INFO[0000] [dialer] Setup tunnel for host [10.0.1.12]   
INFO[0000] [dialer] Setup tunnel for host [10.0.1.217]  
INFO[0000] [dialer] Setup tunnel for host [10.0.1.205]  
FATA[0000] Unsupported Docker version found [27.3.1] on host [10.0.1.217], supported versions are [1.13.x 17.03.x 17.06.x 17.09.x 18.06.x 18.09.x 19.03.x 20.10.x 23.0.x 24.0.x 25.0.x 26.0.x 26.1.x 27.0.x 27.1.x] 


<!-- rke up --config cluster.yaml
INFO[0000] Running RKE version: v1.6.2                  
INFO[0000] Initiating Kubernetes cluster                
INFO[0000] [dialer] Setup tunnel for host [10.0.1.38]   
INFO[0000] [dialer] Setup tunnel for host [10.0.1.17]   
INFO[0000] [dialer] Setup tunnel for host [10.0.1.241]  
WARN[0009] Failed to set up SSH tunneling for host [10.0.1.17]: Can't retrieve Docker Info: error during connect: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/info": Failed to dial ssh using address [10.0.1.17:22]: dial tcp 10.0.1.17:22: connect: no route to host 
WARN[0009] Removing host [10.0.1.17] from node lists    
INFO[0009] Finding container [cluster-state-deployer] on host [10.0.1.241], try #1 
INFO[0009] Pulling image [rancher/rke-tools:v0.1.102] on host [10.0.1.241], try #1 
INFO[0021] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0023] Starting container [cluster-state-deployer] on host [10.0.1.241], try #1 
INFO[0023] [state] Successfully started [cluster-state-deployer] container on host [10.0.1.241] 
INFO[0023] Finding container [cluster-state-deployer] on host [10.0.1.38], try #1 
INFO[0023] Pulling image [rancher/rke-tools:v0.1.102] on host [10.0.1.38], try #1 
INFO[0034] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0037] Starting container [cluster-state-deployer] on host [10.0.1.38], try #1 
INFO[0037] [state] Successfully started [cluster-state-deployer] container on host [10.0.1.38] 
INFO[0037] [certificates] Generating CA kubernetes certificates 
INFO[0037] [certificates] Generating Kubernetes API server aggregation layer requestheader client CA certificates 
INFO[0037] [certificates] GenerateServingCertificate is disabled, checking if there are unused kubelet certificates 
INFO[0037] [certificates] Generating Kubernetes API server certificates 
INFO[0038] [certificates] Generating Service account token key 
INFO[0038] [certificates] Generating Kube Controller certificates 
INFO[0039] [certificates] Generating Kube Scheduler certificates 
INFO[0039] [certificates] Generating Kube Proxy certificates 
INFO[0040] [certificates] Generating Node certificate   
INFO[0040] [certificates] Generating admin certificates and kubeconfig 
INFO[0040] [certificates] Generating Kubernetes API server proxy client certificates 
INFO[0040] [certificates] Generating kube-etcd-10-0-1-241 certificate and key 
INFO[0040] [certificates] Generating kube-etcd-10-0-1-38 certificate and key 
INFO[0041] Successfully Deployed state file at [./cluster.rkestate] 
INFO[0041] Building Kubernetes cluster                  
INFO[0041] [dialer] Setup tunnel for host [10.0.1.241]  
INFO[0041] [dialer] Setup tunnel for host [10.0.1.38]   
INFO[0041] [network] Deploying port listener containers 
INFO[0041] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0041] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0041] Starting container [rke-etcd-port-listener] on host [10.0.1.241], try #1 
INFO[0041] Starting container [rke-etcd-port-listener] on host [10.0.1.38], try #1 
INFO[0041] [network] Successfully started [rke-etcd-port-listener] container on host [10.0.1.241] 
INFO[0042] [network] Successfully started [rke-etcd-port-listener] container on host [10.0.1.38] 
INFO[0042] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0042] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0042] Starting container [rke-cp-port-listener] on host [10.0.1.38], try #1 
INFO[0042] Starting container [rke-cp-port-listener] on host [10.0.1.241], try #1 
INFO[0042] [network] Successfully started [rke-cp-port-listener] container on host [10.0.1.38] 
INFO[0042] [network] Successfully started [rke-cp-port-listener] container on host [10.0.1.241] 
INFO[0042] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0042] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0042] Starting container [rke-worker-port-listener] on host [10.0.1.38], try #1 
INFO[0042] Starting container [rke-worker-port-listener] on host [10.0.1.241], try #1 
INFO[0043] [network] Successfully started [rke-worker-port-listener] container on host [10.0.1.38] 
INFO[0043] [network] Successfully started [rke-worker-port-listener] container on host [10.0.1.241] 
INFO[0043] [network] Port listener containers deployed successfully 
INFO[0043] [network] Running etcd <-> etcd port checks  
INFO[0043] [network] Checking if host [10.0.1.241] can connect to host(s) [10.0.1.241 10.0.1.38] on port(s) [2379 2380], try #1 
INFO[0043] [network] Checking if host [10.0.1.38] can connect to host(s) [10.0.1.241 10.0.1.38] on port(s) [2379 2380], try #1 
INFO[0043] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0043] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0043] Starting container [rke-port-checker] on host [10.0.1.38], try #1 
INFO[0043] Starting container [rke-port-checker] on host [10.0.1.241], try #1 
INFO[0043] [network] Successfully started [rke-port-checker] container on host [10.0.1.241] 
INFO[0043] [network] Successfully started [rke-port-checker] container on host [10.0.1.38] 
INFO[0043] Removing container [rke-port-checker] on host [10.0.1.38], try #1 
INFO[0044] Removing container [rke-port-checker] on host [10.0.1.241], try #1 
INFO[0044] [network] Running control plane -> etcd port checks 
INFO[0044] [network] Checking if host [10.0.1.241] can connect to host(s) [10.0.1.241 10.0.1.38] on port(s) [2379], try #1 
INFO[0044] [network] Checking if host [10.0.1.38] can connect to host(s) [10.0.1.241 10.0.1.38] on port(s) [2379], try #1 
INFO[0044] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0044] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0045] Starting container [rke-port-checker] on host [10.0.1.38], try #1 
INFO[0045] Starting container [rke-port-checker] on host [10.0.1.241], try #1 
INFO[0045] [network] Successfully started [rke-port-checker] container on host [10.0.1.38] 
INFO[0045] [network] Successfully started [rke-port-checker] container on host [10.0.1.241] 
INFO[0045] Removing container [rke-port-checker] on host [10.0.1.38], try #1 
INFO[0045] Removing container [rke-port-checker] on host [10.0.1.241], try #1 
INFO[0045] [network] Running control plane -> worker port checks 
INFO[0045] [network] Checking if host [10.0.1.241] can connect to host(s) [10.0.1.241 10.0.1.38] on port(s) [10250], try #1 
INFO[0045] [network] Checking if host [10.0.1.38] can connect to host(s) [10.0.1.241 10.0.1.38] on port(s) [10250], try #1 
INFO[0045] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0045] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0045] Starting container [rke-port-checker] on host [10.0.1.38], try #1 
INFO[0045] Starting container [rke-port-checker] on host [10.0.1.241], try #1 
INFO[0045] [network] Successfully started [rke-port-checker] container on host [10.0.1.38] 
INFO[0045] [network] Successfully started [rke-port-checker] container on host [10.0.1.241] 
INFO[0045] Removing container [rke-port-checker] on host [10.0.1.38], try #1 
INFO[0045] Removing container [rke-port-checker] on host [10.0.1.241], try #1 
INFO[0045] [network] Running workers -> control plane port checks 
INFO[0045] [network] Checking if host [10.0.1.241] can connect to host(s) [10.0.1.241 10.0.1.38] on port(s) [6443], try #1 
INFO[0045] [network] Checking if host [10.0.1.38] can connect to host(s) [10.0.1.241 10.0.1.38] on port(s) [6443], try #1 
INFO[0045] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0045] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0046] Starting container [rke-port-checker] on host [10.0.1.38], try #1 
INFO[0046] Starting container [rke-port-checker] on host [10.0.1.241], try #1 
INFO[0046] [network] Successfully started [rke-port-checker] container on host [10.0.1.38] 
INFO[0046] Removing container [rke-port-checker] on host [10.0.1.38], try #1 
INFO[0046] [network] Successfully started [rke-port-checker] container on host [10.0.1.241] 
INFO[0046] Removing container [rke-port-checker] on host [10.0.1.241], try #1 
INFO[0046] [network] Checking KubeAPI port Control Plane hosts 
INFO[0046] [network] Removing port listener containers  
INFO[0046] Removing container [rke-etcd-port-listener] on host [10.0.1.38], try #1 
INFO[0046] Removing container [rke-etcd-port-listener] on host [10.0.1.241], try #1 
INFO[0046] [remove/rke-etcd-port-listener] Successfully removed container on host [10.0.1.38] 
INFO[0046] [remove/rke-etcd-port-listener] Successfully removed container on host [10.0.1.241] 
INFO[0046] Removing container [rke-cp-port-listener] on host [10.0.1.241], try #1 
INFO[0046] Removing container [rke-cp-port-listener] on host [10.0.1.38], try #1 
INFO[0046] [remove/rke-cp-port-listener] Successfully removed container on host [10.0.1.38] 
INFO[0046] [remove/rke-cp-port-listener] Successfully removed container on host [10.0.1.241] 
INFO[0046] Removing container [rke-worker-port-listener] on host [10.0.1.38], try #1 
INFO[0046] Removing container [rke-worker-port-listener] on host [10.0.1.241], try #1 
INFO[0047] [remove/rke-worker-port-listener] Successfully removed container on host [10.0.1.241] 
INFO[0047] [remove/rke-worker-port-listener] Successfully removed container on host [10.0.1.38] 
INFO[0047] [network] Port listener containers removed successfully 
INFO[0047] [certificates] Deploying kubernetes certificates to Cluster nodes 
INFO[0047] Finding container [cert-deployer] on host [10.0.1.241], try #1 
INFO[0047] Finding container [cert-deployer] on host [10.0.1.38], try #1 
INFO[0047] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0047] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0047] Starting container [cert-deployer] on host [10.0.1.241], try #1 
INFO[0047] Starting container [cert-deployer] on host [10.0.1.38], try #1 
INFO[0047] Finding container [cert-deployer] on host [10.0.1.241], try #1 
INFO[0047] Finding container [cert-deployer] on host [10.0.1.38], try #1 
INFO[0052] Finding container [cert-deployer] on host [10.0.1.241], try #1 
INFO[0052] Removing container [cert-deployer] on host [10.0.1.241], try #1 
INFO[0052] Finding container [cert-deployer] on host [10.0.1.38], try #1 
INFO[0052] Removing container [cert-deployer] on host [10.0.1.38], try #1 
INFO[0052] [reconcile] Rebuilding and updating local kube config 
INFO[0052] Successfully Deployed local admin kubeconfig at [./kube_config_cluster.yaml] 
WARN[0052] [reconcile] host [10.0.1.241] is a control plane node without reachable Kubernetes API endpoint in the cluster 
INFO[0052] Successfully Deployed local admin kubeconfig at [./kube_config_cluster.yaml] 
WARN[0052] [reconcile] host [10.0.1.38] is a control plane node without reachable Kubernetes API endpoint in the cluster 
WARN[0052] [reconcile] no control plane node with reachable Kubernetes API endpoint in the cluster found 
INFO[0052] [certificates] Successfully deployed kubernetes certificates to Cluster nodes 
INFO[0052] [file-deploy] Deploying file [/etc/kubernetes/admission.yaml] to node [10.0.1.241] 
INFO[0052] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0052] Starting container [file-deployer] on host [10.0.1.241], try #1 
INFO[0053] Successfully started [file-deployer] container on host [10.0.1.241] 
INFO[0053] Waiting for [file-deployer] container to exit on host [10.0.1.241] 
INFO[0053] Waiting for [file-deployer] container to exit on host [10.0.1.241] 
INFO[0053] Container [file-deployer] is still running on host [10.0.1.241]: stderr: [], stdout: [] 
INFO[0054] Removing container [file-deployer] on host [10.0.1.241], try #1 
INFO[0054] [remove/file-deployer] Successfully removed container on host [10.0.1.241] 
INFO[0054] [file-deploy] Deploying file [/etc/kubernetes/admission.yaml] to node [10.0.1.38] 
INFO[0054] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0054] Starting container [file-deployer] on host [10.0.1.38], try #1 
INFO[0054] Successfully started [file-deployer] container on host [10.0.1.38] 
INFO[0054] Waiting for [file-deployer] container to exit on host [10.0.1.38] 
INFO[0054] Waiting for [file-deployer] container to exit on host [10.0.1.38] 
INFO[0054] Container [file-deployer] is still running on host [10.0.1.38]: stderr: [], stdout: [] 
INFO[0055] Removing container [file-deployer] on host [10.0.1.38], try #1 
INFO[0055] [remove/file-deployer] Successfully removed container on host [10.0.1.38] 
INFO[0055] [/etc/kubernetes/admission.yaml] Successfully deployed admission control config to Cluster control nodes 
INFO[0055] [file-deploy] Deploying file [/etc/kubernetes/audit-policy.yaml] to node [10.0.1.38] 
INFO[0055] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0056] Starting container [file-deployer] on host [10.0.1.38], try #1 
INFO[0056] Successfully started [file-deployer] container on host [10.0.1.38] 
INFO[0056] Waiting for [file-deployer] container to exit on host [10.0.1.38] 
INFO[0056] Waiting for [file-deployer] container to exit on host [10.0.1.38] 
INFO[0056] Container [file-deployer] is still running on host [10.0.1.38]: stderr: [], stdout: [] 
INFO[0057] Removing container [file-deployer] on host [10.0.1.38], try #1 
INFO[0057] [remove/file-deployer] Successfully removed container on host [10.0.1.38] 
INFO[0057] [file-deploy] Deploying file [/etc/kubernetes/audit-policy.yaml] to node [10.0.1.241] 
INFO[0057] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0057] Starting container [file-deployer] on host [10.0.1.241], try #1 
INFO[0058] Successfully started [file-deployer] container on host [10.0.1.241] 
INFO[0058] Waiting for [file-deployer] container to exit on host [10.0.1.241] 
INFO[0058] Waiting for [file-deployer] container to exit on host [10.0.1.241] 
INFO[0058] Container [file-deployer] is still running on host [10.0.1.241]: stderr: [], stdout: [] 
INFO[0059] Removing container [file-deployer] on host [10.0.1.241], try #1 
INFO[0059] [remove/file-deployer] Successfully removed container on host [10.0.1.241] 
INFO[0059] [/etc/kubernetes/audit-policy.yaml] Successfully deployed audit policy file to Cluster control nodes 
INFO[0059] [reconcile] Reconciling cluster state        
INFO[0059] [reconcile] This is newly generated cluster  
INFO[0059] Pre-pulling kubernetes images                
INFO[0059] Pulling image [rancher/hyperkube:v1.30.4-rancher1] on host [10.0.1.241], try #1 
INFO[0059] Pulling image [rancher/hyperkube:v1.30.4-rancher1] on host [10.0.1.38], try #1 
INFO[0088] Image [rancher/hyperkube:v1.30.4-rancher1] exists on host [10.0.1.38] 
INFO[0088] Pulling image [rancher/mirrored-pause:3.7] on host [10.0.1.38], try #1 
INFO[0088] Image [rancher/hyperkube:v1.30.4-rancher1] exists on host [10.0.1.241] 
INFO[0088] Pulling image [rancher/mirrored-pause:3.7] on host [10.0.1.241], try #1 
INFO[0093] Image [rancher/mirrored-pause:3.7] exists on host [10.0.1.38] 
INFO[0093] Image [rancher/mirrored-pause:3.7] exists on host [10.0.1.241] 
INFO[0093] Kubernetes images pulled successfully        
INFO[0093] [etcd] Building up etcd plane..              
INFO[0093] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0097] Starting container [etcd-fix-perm] on host [10.0.1.241], try #1 
INFO[0097] Successfully started [etcd-fix-perm] container on host [10.0.1.241] 
INFO[0097] Waiting for [etcd-fix-perm] container to exit on host [10.0.1.241] 
INFO[0097] Waiting for [etcd-fix-perm] container to exit on host [10.0.1.241] 
INFO[0097] Removing container [etcd-fix-perm] on host [10.0.1.241], try #1 
INFO[0098] [remove/etcd-fix-perm] Successfully removed container on host [10.0.1.241] 
INFO[0098] Pulling image [rancher/mirrored-coreos-etcd:v3.5.12] on host [10.0.1.241], try #1 
INFO[0106] Image [rancher/mirrored-coreos-etcd:v3.5.12] exists on host [10.0.1.241] 
INFO[0106] Starting container [etcd] on host [10.0.1.241], try #1 
INFO[0106] [etcd] Successfully started [etcd] container on host [10.0.1.241] 
INFO[0106] [etcd] Running rolling snapshot container [etcd-rolling-snapshots] on host [10.0.1.241] 
INFO[0106] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0107] Starting container [etcd-rolling-snapshots] on host [10.0.1.241], try #1 
INFO[0107] [etcd] Successfully started [etcd-rolling-snapshots] container on host [10.0.1.241] 
INFO[0112] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0112] Starting container [rke-bundle-cert] on host [10.0.1.241], try #1 
INFO[0112] [certificates] Successfully started [rke-bundle-cert] container on host [10.0.1.241] 
INFO[0112] Waiting for [rke-bundle-cert] container to exit on host [10.0.1.241] 
INFO[0113] [certificates] successfully saved certificate bundle [/opt/rke/etcd-snapshots//pki.bundle.tar.gz] on host [10.0.1.241] 
INFO[0113] Removing container [rke-bundle-cert] on host [10.0.1.241], try #1 
INFO[0113] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0113] Starting container [rke-log-linker] on host [10.0.1.241], try #1 
INFO[0113] [etcd] Successfully started [rke-log-linker] container on host [10.0.1.241] 
INFO[0113] Removing container [rke-log-linker] on host [10.0.1.241], try #1 
INFO[0113] [remove/rke-log-linker] Successfully removed container on host [10.0.1.241] 
INFO[0113] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0114] Starting container [rke-log-linker] on host [10.0.1.241], try #1 
INFO[0114] [etcd] Successfully started [rke-log-linker] container on host [10.0.1.241] 
INFO[0114] Removing container [rke-log-linker] on host [10.0.1.241], try #1 
INFO[0114] [remove/rke-log-linker] Successfully removed container on host [10.0.1.241] 
INFO[0114] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0118] Starting container [etcd-fix-perm] on host [10.0.1.38], try #1 
INFO[0118] Successfully started [etcd-fix-perm] container on host [10.0.1.38] 
INFO[0118] Waiting for [etcd-fix-perm] container to exit on host [10.0.1.38] 
INFO[0118] Waiting for [etcd-fix-perm] container to exit on host [10.0.1.38] 
INFO[0119] Removing container [etcd-fix-perm] on host [10.0.1.38], try #1 
INFO[0119] [remove/etcd-fix-perm] Successfully removed container on host [10.0.1.38] 
INFO[0119] Pulling image [rancher/mirrored-coreos-etcd:v3.5.12] on host [10.0.1.38], try #1 
INFO[0126] Image [rancher/mirrored-coreos-etcd:v3.5.12] exists on host [10.0.1.38] 
INFO[0126] Starting container [etcd] on host [10.0.1.38], try #1 
INFO[0127] [etcd] Successfully started [etcd] container on host [10.0.1.38] 
INFO[0127] [etcd] Running rolling snapshot container [etcd-rolling-snapshots] on host [10.0.1.38] 
INFO[0127] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0127] Starting container [etcd-rolling-snapshots] on host [10.0.1.38], try #1 
INFO[0127] [etcd] Successfully started [etcd-rolling-snapshots] container on host [10.0.1.38] 
INFO[0132] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0132] Starting container [rke-bundle-cert] on host [10.0.1.38], try #1 
INFO[0133] [certificates] Successfully started [rke-bundle-cert] container on host [10.0.1.38] 
INFO[0133] Waiting for [rke-bundle-cert] container to exit on host [10.0.1.38] 
INFO[0133] [certificates] successfully saved certificate bundle [/opt/rke/etcd-snapshots//pki.bundle.tar.gz] on host [10.0.1.38] 
INFO[0133] Removing container [rke-bundle-cert] on host [10.0.1.38], try #1 
INFO[0133] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0133] Starting container [rke-log-linker] on host [10.0.1.38], try #1 
INFO[0133] [etcd] Successfully started [rke-log-linker] container on host [10.0.1.38] 
INFO[0133] Removing container [rke-log-linker] on host [10.0.1.38], try #1 
INFO[0133] [remove/rke-log-linker] Successfully removed container on host [10.0.1.38] 
INFO[0133] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0134] Starting container [rke-log-linker] on host [10.0.1.38], try #1 
INFO[0134] [etcd] Successfully started [rke-log-linker] container on host [10.0.1.38] 
INFO[0134] Removing container [rke-log-linker] on host [10.0.1.38], try #1 
INFO[0134] [remove/rke-log-linker] Successfully removed container on host [10.0.1.38] 
INFO[0134] [etcd] Successfully started etcd plane.. Checking etcd cluster health 
INFO[0134] [etcd] etcd host [10.0.1.241] reported healthy=true 
INFO[0134] [controlplane] Building up Controller Plane.. 
INFO[0134] Finding container [service-sidekick] on host [10.0.1.38], try #1 
INFO[0134] Finding container [service-sidekick] on host [10.0.1.241], try #1 
INFO[0134] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0134] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0135] Image [rancher/hyperkube:v1.30.4-rancher1] exists on host [10.0.1.38] 
INFO[0135] Image [rancher/hyperkube:v1.30.4-rancher1] exists on host [10.0.1.241] 
INFO[0135] Starting container [kube-apiserver] on host [10.0.1.38], try #1 
INFO[0135] Starting container [kube-apiserver] on host [10.0.1.241], try #1 
INFO[0135] [controlplane] Successfully started [kube-apiserver] container on host [10.0.1.38] 
INFO[0135] [healthcheck] Start Healthcheck on service [kube-apiserver] on host [10.0.1.38] 
INFO[0135] [controlplane] Successfully started [kube-apiserver] container on host [10.0.1.241] 
INFO[0135] [healthcheck] Start Healthcheck on service [kube-apiserver] on host [10.0.1.241] 
INFO[0142] [healthcheck] service [kube-apiserver] on host [10.0.1.38] is healthy 
INFO[0142] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0142] [healthcheck] service [kube-apiserver] on host [10.0.1.241] is healthy 
INFO[0142] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0142] Starting container [rke-log-linker] on host [10.0.1.38], try #1 
INFO[0142] Starting container [rke-log-linker] on host [10.0.1.241], try #1 
INFO[0142] [controlplane] Successfully started [rke-log-linker] container on host [10.0.1.241] 
INFO[0142] [controlplane] Successfully started [rke-log-linker] container on host [10.0.1.38] 
INFO[0142] Removing container [rke-log-linker] on host [10.0.1.241], try #1 
INFO[0142] Removing container [rke-log-linker] on host [10.0.1.38], try #1 
INFO[0142] [remove/rke-log-linker] Successfully removed container on host [10.0.1.241] 
INFO[0142] [remove/rke-log-linker] Successfully removed container on host [10.0.1.38] 
INFO[0142] Image [rancher/hyperkube:v1.30.4-rancher1] exists on host [10.0.1.241] 
INFO[0142] Image [rancher/hyperkube:v1.30.4-rancher1] exists on host [10.0.1.38] 
INFO[0143] Starting container [kube-controller-manager] on host [10.0.1.241], try #1 
INFO[0143] Starting container [kube-controller-manager] on host [10.0.1.38], try #1 
INFO[0143] [controlplane] Successfully started [kube-controller-manager] container on host [10.0.1.38] 
INFO[0143] [healthcheck] Start Healthcheck on service [kube-controller-manager] on host [10.0.1.38] 
INFO[0143] [controlplane] Successfully started [kube-controller-manager] container on host [10.0.1.241] 
INFO[0143] [healthcheck] Start Healthcheck on service [kube-controller-manager] on host [10.0.1.241] 
INFO[0148] [healthcheck] service [kube-controller-manager] on host [10.0.1.241] is healthy 
INFO[0148] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0148] [healthcheck] service [kube-controller-manager] on host [10.0.1.38] is healthy 
INFO[0148] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0148] Starting container [rke-log-linker] on host [10.0.1.241], try #1 
INFO[0148] Starting container [rke-log-linker] on host [10.0.1.38], try #1 
INFO[0149] [controlplane] Successfully started [rke-log-linker] container on host [10.0.1.241] 
INFO[0149] [controlplane] Successfully started [rke-log-linker] container on host [10.0.1.38] 
INFO[0149] Removing container [rke-log-linker] on host [10.0.1.38], try #1 
INFO[0149] Removing container [rke-log-linker] on host [10.0.1.241], try #1 
INFO[0149] [remove/rke-log-linker] Successfully removed container on host [10.0.1.38] 
INFO[0149] [remove/rke-log-linker] Successfully removed container on host [10.0.1.241] 
INFO[0149] Image [rancher/hyperkube:v1.30.4-rancher1] exists on host [10.0.1.38] 
INFO[0149] Image [rancher/hyperkube:v1.30.4-rancher1] exists on host [10.0.1.241] 
INFO[0149] Starting container [kube-scheduler] on host [10.0.1.38], try #1 
INFO[0149] Starting container [kube-scheduler] on host [10.0.1.241], try #1 
INFO[0149] [controlplane] Successfully started [kube-scheduler] container on host [10.0.1.241] 
INFO[0149] [healthcheck] Start Healthcheck on service [kube-scheduler] on host [10.0.1.241] 
INFO[0149] [controlplane] Successfully started [kube-scheduler] container on host [10.0.1.38] 
INFO[0149] [healthcheck] Start Healthcheck on service [kube-scheduler] on host [10.0.1.38] 
INFO[0154] [healthcheck] service [kube-scheduler] on host [10.0.1.241] is healthy 
INFO[0154] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0154] [healthcheck] service [kube-scheduler] on host [10.0.1.38] is healthy 
INFO[0154] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0154] Starting container [rke-log-linker] on host [10.0.1.241], try #1 
INFO[0154] Starting container [rke-log-linker] on host [10.0.1.38], try #1 
INFO[0155] [controlplane] Successfully started [rke-log-linker] container on host [10.0.1.38] 
INFO[0155] [controlplane] Successfully started [rke-log-linker] container on host [10.0.1.241] 
INFO[0155] Removing container [rke-log-linker] on host [10.0.1.38], try #1 
INFO[0155] Removing container [rke-log-linker] on host [10.0.1.241], try #1 
INFO[0155] [remove/rke-log-linker] Successfully removed container on host [10.0.1.38] 
INFO[0155] [remove/rke-log-linker] Successfully removed container on host [10.0.1.241] 
INFO[0155] [controlplane] Successfully started Controller Plane.. 
INFO[0155] [authz] Creating rke-job-deployer ServiceAccount 
INFO[0155] [authz] rke-job-deployer ServiceAccount created successfully 
INFO[0155] [authz] Creating system:node ClusterRoleBinding 
INFO[0155] [authz] system:node ClusterRoleBinding created successfully 
INFO[0155] [authz] Creating kube-apiserver proxy ClusterRole and ClusterRoleBinding 
INFO[0155] [authz] kube-apiserver proxy ClusterRole and ClusterRoleBinding created successfully 
INFO[0155] Successfully Deployed state file at [./cluster.rkestate] 
INFO[0155] [state] Saving full cluster state to Kubernetes 
INFO[0155] [worker] Building up Worker Plane..          
INFO[0155] Finding container [service-sidekick] on host [10.0.1.38], try #1 
INFO[0155] Finding container [service-sidekick] on host [10.0.1.241], try #1 
INFO[0155] [sidekick] Sidekick container already created on host [10.0.1.38] 
INFO[0155] [sidekick] Sidekick container already created on host [10.0.1.241] 
INFO[0155] Image [rancher/hyperkube:v1.30.4-rancher1] exists on host [10.0.1.38] 
INFO[0155] Image [rancher/hyperkube:v1.30.4-rancher1] exists on host [10.0.1.241] 
INFO[0155] Starting container [kubelet] on host [10.0.1.38], try #1 
INFO[0155] Starting container [kubelet] on host [10.0.1.241], try #1 
INFO[0155] [worker] Successfully started [kubelet] container on host [10.0.1.38] 
INFO[0155] [healthcheck] Start Healthcheck on service [kubelet] on host [10.0.1.38] 
INFO[0155] [worker] Successfully started [kubelet] container on host [10.0.1.241] 
INFO[0155] [healthcheck] Start Healthcheck on service [kubelet] on host [10.0.1.241] 
INFO[0171] [healthcheck] service [kubelet] on host [10.0.1.241] is healthy 
INFO[0171] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0171] [healthcheck] service [kubelet] on host [10.0.1.38] is healthy 
INFO[0171] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0171] Starting container [rke-log-linker] on host [10.0.1.241], try #1 
INFO[0171] Starting container [rke-log-linker] on host [10.0.1.38], try #1 
INFO[0171] [worker] Successfully started [rke-log-linker] container on host [10.0.1.241] 
INFO[0171] Removing container [rke-log-linker] on host [10.0.1.241], try #1 
INFO[0171] [remove/rke-log-linker] Successfully removed container on host [10.0.1.241] 
INFO[0171] Image [rancher/hyperkube:v1.30.4-rancher1] exists on host [10.0.1.241] 
INFO[0171] Starting container [kube-proxy] on host [10.0.1.241], try #1 
INFO[0171] [worker] Successfully started [rke-log-linker] container on host [10.0.1.38] 
INFO[0171] [worker] Successfully started [kube-proxy] container on host [10.0.1.241] 
INFO[0171] [healthcheck] Start Healthcheck on service [kube-proxy] on host [10.0.1.241] 
INFO[0172] Removing container [rke-log-linker] on host [10.0.1.38], try #1 
INFO[0172] [remove/rke-log-linker] Successfully removed container on host [10.0.1.38] 
INFO[0172] Image [rancher/hyperkube:v1.30.4-rancher1] exists on host [10.0.1.38] 
INFO[0172] Starting container [kube-proxy] on host [10.0.1.38], try #1 
INFO[0172] [healthcheck] service [kube-proxy] on host [10.0.1.241] is healthy 
INFO[0172] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0172] [worker] Successfully started [kube-proxy] container on host [10.0.1.38] 
INFO[0172] [healthcheck] Start Healthcheck on service [kube-proxy] on host [10.0.1.38] 
INFO[0172] [healthcheck] service [kube-proxy] on host [10.0.1.38] is healthy 
INFO[0172] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0172] Starting container [rke-log-linker] on host [10.0.1.241], try #1 
INFO[0172] Starting container [rke-log-linker] on host [10.0.1.38], try #1 
INFO[0172] [worker] Successfully started [rke-log-linker] container on host [10.0.1.241] 
INFO[0172] Removing container [rke-log-linker] on host [10.0.1.241], try #1 
INFO[0172] [remove/rke-log-linker] Successfully removed container on host [10.0.1.241] 
INFO[0172] [worker] Successfully started [rke-log-linker] container on host [10.0.1.38] 
INFO[0173] Removing container [rke-log-linker] on host [10.0.1.38], try #1 
INFO[0173] [remove/rke-log-linker] Successfully removed container on host [10.0.1.38] 
INFO[0173] [worker] Successfully started Worker Plane.. 
INFO[0173] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.241] 
INFO[0173] Image [rancher/rke-tools:v0.1.102] exists on host [10.0.1.38] 
INFO[0173] Starting container [rke-log-cleaner] on host [10.0.1.38], try #1 
INFO[0173] Starting container [rke-log-cleaner] on host [10.0.1.241], try #1 
INFO[0173] [cleanup] Successfully started [rke-log-cleaner] container on host [10.0.1.38] 
INFO[0173] [cleanup] Successfully started [rke-log-cleaner] container on host [10.0.1.241] 
INFO[0173] Removing container [rke-log-cleaner] on host [10.0.1.38], try #1 
INFO[0173] Removing container [rke-log-cleaner] on host [10.0.1.241], try #1 
INFO[0173] [remove/rke-log-cleaner] Successfully removed container on host [10.0.1.38] 
INFO[0173] [remove/rke-log-cleaner] Successfully removed container on host [10.0.1.241] 
INFO[0173] [sync] Syncing nodes Labels and Taints       
INFO[0173] [sync] Successfully synced nodes Labels and Taints 
INFO[0173] [network] Setting up network plugin: canal   
INFO[0173] [addons] Saving ConfigMap for addon rke-network-plugin to Kubernetes 
INFO[0173] [addons] Successfully saved ConfigMap for addon rke-network-plugin to Kubernetes 
INFO[0173] [addons] Executing deploy job rke-network-plugin 
INFO[0178] [addons] Setting up coredns                  
INFO[0178] [addons] Saving ConfigMap for addon rke-coredns-addon to Kubernetes 
INFO[0178] [addons] Successfully saved ConfigMap for addon rke-coredns-addon to Kubernetes 
INFO[0178] [addons] Executing deploy job rke-coredns-addon 
INFO[0183] [addons] CoreDNS deployed successfully       
INFO[0183] [dns] DNS provider coredns deployed successfully 
INFO[0183] [addons] Setting up Metrics Server           
INFO[0183] [addons] Saving ConfigMap for addon rke-metrics-addon to Kubernetes 
INFO[0183] [addons] Successfully saved ConfigMap for addon rke-metrics-addon to Kubernetes 
INFO[0183] [addons] Executing deploy job rke-metrics-addon 
INFO[0193] [addons] Metrics Server deployed successfully 
INFO[0193] [ingress] Setting up nginx ingress controller 
INFO[0193] [ingress] removing admission batch jobs if they exist 
INFO[0194] [addons] Saving ConfigMap for addon rke-ingress-controller to Kubernetes 
INFO[0194] [addons] Successfully saved ConfigMap for addon rke-ingress-controller to Kubernetes 
INFO[0194] [addons] Executing deploy job rke-ingress-controller 
INFO[0204] [ingress] removing default backend service and deployment if they exist 
INFO[0204] [ingress] ingress controller nginx deployed successfully 
INFO[0204] [addons] Setting up user addons              
INFO[0204] [addons] no user addons defined              
INFO[0204] Finished building Kubernetes cluster successfully  -->


# 2 file generate after rke up _important file
cat clusters.rkestate
cat kube_config_cluster.yaml

# check os cpu
lscpu


# install kubectl
 curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

mkdir .kube
cp -r kube_config_cluster.yaml .kube/config


# kubectl get nodes -o wide
ubuntu@node01:~$ kubectl get nodes -o wide
NAME        STATUS   ROLES                      AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
k8s-node1   Ready    controlplane,etcd,worker   59s   v1.30.4   10.0.1.7      <none>        Ubuntu 22.04.5 LTS   6.8.0-1015-aws   docker://20.10.24
k8s-node2   Ready    controlplane,etcd,worker   15m   v1.30.4   10.0.1.38     <none>        Ubuntu 22.04.5 LTS   6.8.0-1015-aws   docker://20.10.24
k8s-node3   Ready    controlplane,etcd,worker   15m   v1.30.4   10.0.1.241    <none>        Ubuntu 22.04.5 LTS   6.8.0-1015-aws   docker://20.10.24
ubuntu@node01:~$ 


ubuntu@node01:~$ kubectl get all -A
NAMESPACE       NAME                                           READY   STATUS      RESTARTS   AGE
ingress-nginx   pod/nginx-ingress-controller-qbvcm             1/1     Running     0          63s
ingress-nginx   pod/nginx-ingress-controller-rkrxq             1/1     Running     0          15m
ingress-nginx   pod/nginx-ingress-controller-wlldj             1/1     Running     0          15m
kube-system     pod/calico-kube-controllers-79b5bd8b74-nsq46   1/1     Running     0          16m
kube-system     pod/canal-gq8sw                                2/2     Running     0          16m
kube-system     pod/canal-n79rn                                2/2     Running     0          108s
kube-system     pod/canal-zdt6t                                2/2     Running     0          16m
kube-system     pod/coredns-878dbb559-2f842                    1/1     Running     0          58s
kube-system     pod/coredns-878dbb559-hhss6                    1/1     Running     0          15m
kube-system     pod/coredns-autoscaler-7db8ddc8cf-qb52r        1/1     Running     0          16m
kube-system     pod/metrics-server-85f57469bf-wq7q5            1/1     Running     0          16m
kube-system     pod/rke-coredns-addon-deploy-job-ltnd9         0/1     Completed   0          16m
kube-system     pod/rke-ingress-controller-deploy-job-b66ks    0/1     Completed   0          15m
kube-system     pod/rke-metrics-addon-deploy-job-f4kv5         0/1     Completed   0          16m
kube-system     pod/rke-network-plugin-deploy-job-28jtp        0/1     Completed   0          16m

NAMESPACE       NAME                                         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                  AGE
default         service/kubernetes                           ClusterIP   10.43.0.1     <none>        443/TCP                  16m
ingress-nginx   service/ingress-nginx-controller-admission   ClusterIP   10.43.83.77   <none>        443/TCP                  15m
kube-system     service/kube-dns                             ClusterIP   10.43.0.10    <none>        53/UDP,53/TCP,9153/TCP   16m
kube-system     service/metrics-server                       ClusterIP   10.43.27.46   <none>        443/TCP                  16m

NAMESPACE       NAME                                      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
ingress-nginx   daemonset.apps/nginx-ingress-controller   3         3         3       3            3           <none>                   15m
kube-system     daemonset.apps/canal                      3         3         3       3            3           kubernetes.io/os=linux   16m

NAMESPACE     NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/calico-kube-controllers   1/1     1            1           16m
kube-system   deployment.apps/coredns                   2/2     2            2           16m
kube-system   deployment.apps/coredns-autoscaler        1/1     1            1           16m
kube-system   deployment.apps/metrics-server            1/1     1            1           16m

NAMESPACE     NAME                                                 DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/calico-kube-controllers-79b5bd8b74   1         1         1       16m
kube-system   replicaset.apps/coredns-878dbb559                    2         2         2       16m
kube-system   replicaset.apps/coredns-autoscaler-7db8ddc8cf        1         1         1       16m
kube-system   replicaset.apps/metrics-server-85f57469bf            1         1         1       16m

NAMESPACE     NAME                                          STATUS     COMPLETIONS   DURATION   AGE
kube-system   job.batch/rke-coredns-addon-deploy-job        Complete   1/1           4s         16m
kube-system   job.batch/rke-ingress-controller-deploy-job   Complete   1/1           6s         15m
kube-system   job.batch/rke-metrics-addon-deploy-job        Complete   1/1           5s         16m
kube-system   job.batch/rke-network-plugin-deploy-job       Complete   1/1           4s         16m


ubuntu@node01:~$ docker ps | grep etcd
e81265cf182d   rancher/rke-tools:v0.1.102             "/docker-entrypoint.…"   3 minutes ago        Up 3 minutes                  etcd-rolling-snapshots
19b1486a9772   rancher/mirrored-coreos-etcd:v3.5.12   "/usr/local/bin/etcd…"   4 minutes ago        Up 4 minutes                  etcd
ubuntu@node01:~$ 


# download app
wget https://raw.githubusercontent.com/istio/istio/release-1.23/samples/bookinfo/platform/kube/bookinfo.yaml


ubuntu@node01:~$ kubectl create ns prod-deployment
namespace/prod-deployment created
ubuntu@node01:~$ kubectl apply -f bookinfo.yaml -n prod-deployment
service/details created
serviceaccount/bookinfo-details created
deployment.apps/details-v1 created
service/ratings created
serviceaccount/bookinfo-ratings created
deployment.apps/ratings-v1 created
service/reviews created
serviceaccount/bookinfo-reviews created
deployment.apps/reviews-v1 created
deployment.apps/reviews-v2 created
deployment.apps/reviews-v3 created
service/productpage created
serviceaccount/bookinfo-productpage created
deployment.apps/productpage-v1 created
ubuntu@node01:~$ watch kubectl get pods -n prod-deployment


kubectl apply -f https://raw.githubusercontent.com/istio/istio/release-1.23/samples/sleep/sleep.yaml -n prod-deployment


ubuntu@node01:~$ kubectl exec -it sleep-5577c64d7c-z42h6 -n prod-deployment -- sh 
~ $ curl http://productpage:9080/productpage 

<meta charset="utf-8">
<meta http-equiv="X-UA-Compatible" content="IE=edge">
<meta name="viewport" content="width=device-width, initial-scale=1.0">


<title>Simple Bookstore App</title>

<script src="static/tailwind/tailwind.css"></script>
<script type="text/javascript">
  window.addEventListener("DOMContentLoaded", (event) => {
    const dialog = document.querySelector("dialog");
    const showButton = document.querySelector("#sign-in-button");
    const closeButton = document.querySelector("#close-dialog");

    if (showButton) {
      showButton.addEventListener("click", () => {
        dialog.showModal();
      });
    }

    if (closeButton) {
      closeButton.addEventListener("click", () => {
        dialog.close();
      });
    }
  })
</script>



<nav class="bg-gray-800">
  <div class="container mx-auto px-4 sm:px-6 lg:px-8">
    <div class="relative flex h-16 items-center justify-between">
      <a href="#" class="text-white px-3 py-2 text-lg font-medium" aria-current="page">BookInfo Sample</a>
      <div class="absolute inset-y-0 right-0 flex items-center pr-2 sm:static sm:inset-auto sm:ml-6 sm:pr-0">
        
          <button type="button" id="sign-in-button" class="rounded-md bg-blue-600 px-3.5 py-2.5 text-sm font-semibold text-white shadow-sm hover:bg-blue-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-blue-600">
            Sign in
          </button>
        
      </div>
    </div>
  </div>
</nav>

<!-- Sign in dialog -->
<dialog id="dialog" class="w-full sm:w-2/3 lg:w-1/3 border rounded-md shadow-xl">
  <div class="flex min-h-full flex-col justify-center px-6 py-12 lg:px-8">
    <div class="absolute right-0 top-0 hidden pr-4 pt-4 sm:block">
      <button id="close-dialog" type="button" class="rounded-md bg-white text-gray-400 hover:text-gray-500 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2">
        <span class="sr-only">Close</span>
        <svg class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" aria-hidden="true">
          <path stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
        </svg>
      </button>
    </div>
    <div class="sm:mx-auto sm:w-full sm:max-w-sm">
        <svg  class="mx-auto h-24 w-auto" xmlns="http://www.w3.org/2000/svg" version="1.1" viewBox="0 0 320 320"><g id="logo" fill="#466BB0"><polygon id="hull" points="80 250 240 250 140 280 80 250"/><polygon id="mainsail" points="80 240 140 230 140 120 80 240"/><polygon id="headsail" points="150 230 240 240 150 40 150 230"/></g></svg>
        <h2 class="mt-5 text-center text-2xl font-bold leading-9 tracking-tight text-gray-900">Sign in to BookInfo</h2>
    </div>
    <div class="mt-10 sm:mx-auto sm:w-full sm:max-w-sm">
      <form class="space-y-6" method="post" action='login' name="login_form">
        <div>
          <label for="email" class="block text-sm font-medium leading-6 text-gray-900">Username</label>
          <div class="mt-2">
            <input id="username" name="username" required class="block w-full px-3 rounded-md border-0 py-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-blue-600 sm:text-sm sm:leading-6">
          </div>
        </div>
        <div>
          <div class="flex items-center justify-between">
            <label for="password" class="block text-sm font-medium leading-6 text-gray-900">Password</label>
          </div>
          <div class="mt-2">
            <input id="password" name="passwd" type="password" required class="block w-full px-3 rounded-md border-0 py-1.5 text-gray-900 shadow-sm ring-1 ring-inset ring-gray-300 placeholder:text-gray-400 focus:ring-2 focus:ring-inset focus:ring-blue-600 sm:text-sm sm:leading-6">
          </div>
        </div>
        <div>
          <button type="submit" class="flex w-full justify-center rounded-md bg-blue-600 px-3 py-1.5 text-sm font-semibold leading-6 text-white shadow-sm hover:bg-blue-500 focus-visible:outline focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-blue-600">Sign in</button>
        </div>
      </form>
      <p class="mt-10 text-center text-sm text-gray-500">
        Not using Istio yet?
        <a href="https://istio.io" target="_blank" class="font-semibold leading-6 text-blue-600 hover:text-blue-500">Start here</a>
      </p>
    </div>
  </div>
</dialog>

<!-- Book description section -->
<div class="container mt-8 mx-auto px-4 sm:px-6 lg:px-8">
  <h1 class="text-5xl font-bold tracking-tight text-blue-900">The Comedy of Errors</h1>
  <div class="mt-6 max-w-4xl">
    
    <p class="mt-6 text-xl leading-8 text-gray-600"><a href="https://en.wikipedia.org/wiki/The_Comedy_of_Errors">Wikipedia Summary</a>: The Comedy of Errors is one of <b>William Shakespeare's</b> early plays. It is his shortest and one of his most farcical comedies, with a major part of the humour coming from slapstick and mistaken identity, in addition to puns and word play.</p>
    
    <div class="mt-6">
      <a href="https://istio.io" target="_blank" class="text-sm font-semibold leading-6 text-blue-600 hover:text-blue-700">Learn more about Istio <span aria-hidden="true">→</span></a>
    </div>

  </div>
</div>

<!-- Book details table -->
<div class="container mt-8 mx-auto px-4 sm:px-6 lg:px-8">
  <div class="mt-4 py-10">
      <div class="max-w-2xl">
        <div class="flow-root">
          
          <h4 class="text-3xl font-semibold">Book Details</h4>
          <div class="-mx-4 -my-2 overflow-x-auto sm:-mx-6 lg:-mx-8">
            <div class="inline-block min-w-full py-2 align-middle sm:px-6 lg:px-8">
              <table class="min-w-full divide-y divide-gray-300">
                <thead>
                  <tr>
                    <th scope="col" class="whitespace-nowrap py-3.5 pl-4 pr-3 text-left text-sm font-semibold text-gray-900 sm:pl-0">ISBN-10</th>
                    <th scope="col" class="whitespace-nowrap px-2 py-3.5 text-left text-sm font-semibold text-gray-900">Publisher</th>
                    <th scope="col" class="whitespace-nowrap px-2 py-3.5 text-left text-sm font-semibold text-gray-900">Pages</th>
                    <th scope="col" class="whitespace-nowrap px-2 py-3.5 text-left text-sm font-semibold text-gray-900">Type</th>
                    <th scope="col" class="whitespace-nowrap px-2 py-3.5 text-left text-sm font-semibold text-gray-900">Language</th>
                  </tr>
                </thead>
                <tbody class="divide-y divide-gray-200 bg-white">
                  <tr>
                    <td class="whitespace-nowrap py-2 pl-4 pr-3 text-sm text-gray-500 sm:pl-0">1234567890</td>
                    <td class="whitespace-nowrap px-2 py-2 text-sm font-medium text-gray-900">PublisherA</td>
                    <td class="whitespace-nowrap px-2 py-2 text-sm text-gray-900">200</td>
                    <td class="whitespace-nowrap px-2 py-2 text-sm text-gray-500">paperback</td>
                    <td class="whitespace-nowrap px-2 py-2 text-sm text-gray-500">English</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
          
        </div>
      </div>
  </div>
</div>

<!-- Book reviews section -->
<div class="bg-blue-600/5 py-12 mx-auto" >
  <div class="container mx-auto px-4 sm:px-6 lg:px-8>
    <div class="max-w-2xl">
      
      <h4 class="text-3xl font-semibold">Book Reviews</h4>
      <div class="flex flex-col md:flex-row">
        
        <section class="px-6 py-12 sm:py-8 lg:px-8">
          <div class="mx-auto max-w-2xl">
            
            
            <div class="flex gap-x-1 text-black-500">
              
              <svg id="glyphicon glyphicon-star" class="h-5 w-5 flex-none" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
                <path fill-rule="evenodd" d="M10.868 2.884c-.321-.772-1.415-.772-1.736 0l-1.83 4.401-4.753.381c-.833.067-1.171 1.107-.536 1.651l3.62 3.102-1.106 4.637c-.194.813.691 1.456 1.405 1.02L10 15.591l4.069 2.485c.713.436 1.598-.207 1.404-1.02l-1.106-4.637 3.62-3.102c.635-.544.297-1.584-.536-1.65l-4.752-.382-1.831-4.401z" clip-rule="evenodd" />
              </svg>
              
              <svg id="glyphicon glyphicon-star" class="h-5 w-5 flex-none" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
                <path fill-rule="evenodd" d="M10.868 2.884c-.321-.772-1.415-.772-1.736 0l-1.83 4.401-4.753.381c-.833.067-1.171 1.107-.536 1.651l3.62 3.102-1.106 4.637c-.194.813.691 1.456 1.405 1.02L10 15.591l4.069 2.485c.713.436 1.598-.207 1.404-1.02l-1.106-4.637 3.62-3.102c.635-.544.297-1.584-.536-1.65l-4.752-.382-1.831-4.401z" clip-rule="evenodd" />
              </svg>
              
              <svg id="glyphicon glyphicon-star" class="h-5 w-5 flex-none" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
                <path fill-rule="evenodd" d="M10.868 2.884c-.321-.772-1.415-.772-1.736 0l-1.83 4.401-4.753.381c-.833.067-1.171 1.107-.536 1.651l3.62 3.102-1.106 4.637c-.194.813.691 1.456 1.405 1.02L10 15.591l4.069 2.485c.713.436 1.598-.207 1.404-1.02l-1.106-4.637 3.62-3.102c.635-.544.297-1.584-.536-1.65l-4.752-.382-1.831-4.401z" clip-rule="evenodd" />
              </svg>
              
              <svg id="glyphicon glyphicon-star" class="h-5 w-5 flex-none" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
                <path fill-rule="evenodd" d="M10.868 2.884c-.321-.772-1.415-.772-1.736 0l-1.83 4.401-4.753.381c-.833.067-1.171 1.107-.536 1.651l3.62 3.102-1.106 4.637c-.194.813.691 1.456 1.405 1.02L10 15.591l4.069 2.485c.713.436 1.598-.207 1.404-1.02l-1.106-4.637 3.62-3.102c.635-.544.297-1.584-.536-1.65l-4.752-.382-1.831-4.401z" clip-rule="evenodd" />
              </svg>
              
              <svg id="glyphicon glyphicon-star" class="h-5 w-5 flex-none" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
                <path fill-rule="evenodd" d="M10.868 2.884c-.321-.772-1.415-.772-1.736 0l-1.83 4.401-4.753.381c-.833.067-1.171 1.107-.536 1.651l3.62 3.102-1.106 4.637c-.194.813.691 1.456 1.405 1.02L10 15.591l4.069 2.485c.713.436 1.598-.207 1.404-1.02l-1.106-4.637 3.62-3.102c.635-.544.297-1.584-.536-1.65l-4.752-.382-1.831-4.401z" clip-rule="evenodd" />
              </svg>
              
              
            </div>
            
            
            <blockquote class="mt-10 text-xl font-semibold leading-8 tracking-tight text-gray-900 sm:text-2xl sm:leading-9">
              <p>"An extremely entertaining play by Shakespeare. The slapstick humour is refreshing!"</p>
            </blockquote>
            <div class="mt-4 flex items-center gap-x-6">
              <img class="h-16 w-16 rounded-full bg-gray-50" src="/static/img/izzy.png" alt="Izzy">
  
              <div class="text-sm leading-6">
                <div class="font-semibold text-gray-900">Reviewer1</div>
                <div class="mt-0.5 text-gray-600 font-mono">Reviews served by: 
                  reviews-v2-6f85cb9b7c-2hf7c
                  
                </div>
              </div>
            </div>
        </section>
        
        <section class="px-6 py-12 sm:py-8 lg:px-8">
          <div class="mx-auto max-w-2xl">
            
            
            <div class="flex gap-x-1 text-black-500">
              
              <svg id="glyphicon glyphicon-star" class="h-5 w-5 flex-none" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
                <path fill-rule="evenodd" d="M10.868 2.884c-.321-.772-1.415-.772-1.736 0l-1.83 4.401-4.753.381c-.833.067-1.171 1.107-.536 1.651l3.62 3.102-1.106 4.637c-.194.813.691 1.456 1.405 1.02L10 15.591l4.069 2.485c.713.436 1.598-.207 1.404-1.02l-1.106-4.637 3.62-3.102c.635-.544.297-1.584-.536-1.65l-4.752-.382-1.831-4.401z" clip-rule="evenodd" />
              </svg>
              
              <svg id="glyphicon glyphicon-star" class="h-5 w-5 flex-none" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
                <path fill-rule="evenodd" d="M10.868 2.884c-.321-.772-1.415-.772-1.736 0l-1.83 4.401-4.753.381c-.833.067-1.171 1.107-.536 1.651l3.62 3.102-1.106 4.637c-.194.813.691 1.456 1.405 1.02L10 15.591l4.069 2.485c.713.436 1.598-.207 1.404-1.02l-1.106-4.637 3.62-3.102c.635-.544.297-1.584-.536-1.65l-4.752-.382-1.831-4.401z" clip-rule="evenodd" />
              </svg>
              
              <svg id="glyphicon glyphicon-star" class="h-5 w-5 flex-none" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
                <path fill-rule="evenodd" d="M10.868 2.884c-.321-.772-1.415-.772-1.736 0l-1.83 4.401-4.753.381c-.833.067-1.171 1.107-.536 1.651l3.62 3.102-1.106 4.637c-.194.813.691 1.456 1.405 1.02L10 15.591l4.069 2.485c.713.436 1.598-.207 1.404-1.02l-1.106-4.637 3.62-3.102c.635-.544.297-1.584-.536-1.65l-4.752-.382-1.831-4.401z" clip-rule="evenodd" />
              </svg>
              
              <svg id="glyphicon glyphicon-star" class="h-5 w-5 flex-none" viewBox="0 0 20 20" fill="currentColor" aria-hidden="true">
                <path fill-rule="evenodd" d="M10.868 2.884c-.321-.772-1.415-.772-1.736 0l-1.83 4.401-4.753.381c-.833.067-1.171 1.107-.536 1.651l3.62 3.102-1.106 4.637c-.194.813.691 1.456 1.405 1.02L10 15.591l4.069 2.485c.713.436 1.598-.207 1.404-1.02l-1.106-4.637 3.62-3.102c.635-.544.297-1.584-.536-1.65l-4.752-.382-1.831-4.401z" clip-rule="evenodd" />
              </svg>
              
              
              <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="h-5 w-5 flex-none">
                <path stroke-linecap="round" stroke-linejoin="round" d="M11.48 3.499a.562.562 0 0 1 1.04 0l2.125 5.111a.563.563 0 0 0 .475.345l5.518.442c.499.04.701.663.321.988l-4.204 3.602a.563.563 0 0 0-.182.557l1.285 5.385a.562.562 0 0 1-.84.61l-4.725-2.885a.562.562 0 0 0-.586 0L6.982 20.54a.562.562 0 0 1-.84-.61l1.285-5.386a.562.562 0 0 0-.182-.557l-4.204-3.602a.562.562 0 0 1 .321-.988l5.518-.442a.563.563 0 0 0 .475-.345L11.48 3.5Z" />
              </svg>
              
            </div>
            
            
            <blockquote class="mt-10 text-xl font-semibold leading-8 tracking-tight text-gray-900 sm:text-2xl sm:leading-9">
              <p>"Absolutely fun and entertaining. The play lacks thematic depth when compared to other plays by Shakespeare."</p>
            </blockquote>
            <div class="mt-4 flex items-center gap-x-6">
              <img class="h-16 w-16 rounded-full bg-gray-50" src="/static/img/izzy.png" alt="Izzy">
  
              <div class="text-sm leading-6">
                <div class="font-semibold text-gray-900">Reviewer2</div>
                <div class="mt-0.5 text-gray-600 font-mono">Reviews served by: 
                  reviews-v2-6f85cb9b7c-2hf7c
                  
                </div>
              </div>
            </div>
        </section>
        
      </div>
      
    </div>
    </div>
  </div>
</div>


# 
ubuntu@node01:~$ cat service-nodeport-productpage.yaml 
apiVersion: v1
kind: Service
metadata:
  name: productpage-np
  labels:
    app: productpage
    service: productpage
spec:
  type: NodePort
  ports:
  - port: 9080
    name: http
    nodePort: 30000
  selector:
    app: productpage

# intsll ngix 
sudo apt install nginx
nginx -v 
sudo systemctl status nginx


# nginx config
stream {
    server {
        listen 8080;
        #TCP traffic will be forwarded to the "stream_backend" upstream group
        proxy_timeout 10m;
        proxy_connect_timeout 5m;
        proxy_pass product-page-server;
    }
    
    upstream product-page-server {
       least_conn;
        server 10.0.1.7:3000 max_fails=3 fail_timeout=5s;
        server 10.0.1.241:3000 max_fails=3 fail_timeout=5s;
        server 10.0.1.38:80 max_fails=3 fail_timeout=5s;
    }
    
}

# instal istioctl 
curl -L https://istio.io/downloadIstio | sh -

cd istio-1.23.2

export PATH=$PWD/bin:$PATH


# install istio
ubuntu@node01:~/istio-1.23.2$ cat istio.yaml 
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  namespace: istio-system
  name: istiocontrolplane
spec:
  components:
    ingressGateways:
      - name: istio-ingressgateway
        enabled: true
        k8s:
          affinity:
            nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                  - matchExpressions:
                      - key: kubernetes.io/hostname
                        operator: In
                        values:
                          - k8s-node01
                          - k8s-node02
                          - k8s-node03
          service:
            type: NodePort
            #externalTrafficPolicy: Local
            ports:
              - name: status-port
                port: 15021
                protocol: TCP
              - name: tls-istiod
                port: 15012
                protocol: TCP
              - name: tls
                port: 15443
                nodePort: 31371
                protocol: TCP
              - name: http2
                port: 80
                nodePort: 30080
                protocol: TCP
              - name: https
                port: 443
                nodePort: 30100
                protocol: TCP
              - name: devops-test
                port: 31010
                nodePort: 31010
                protocol: TCP
  profile: demo
  meshConfig:
    defaultConfig:
      gatewayTopology:
        numTrustedProxies: 2
    accessLogFile: /dev/stdout
    enableTracing: true    ingressGateways:
      - name: istio-ingressgateway
        enabled: true
        k8s:
          affinity:
            nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                  - matchExpressions:
                      - key: kubernetes.io/hostname
                        operator: In
                        values:
                          - k8s-node01
                          - k8s-node02
                          - k8s-node03
          service:
            type: NodePort
            #externalTrafficPolicy: Local
            ports:
              - name: status-port
                port: 15021
                protocol: TCP
              - name: tls-istiod
                port: 15012
                protocol: TCP
              - name: tls
                port: 15443
                nodePort: 31371
                protocol: TCP
              - name: http2
                port: 80
                nodePort: 30080
                protocol: TCP
              - name: https
                port: 443
                nodePort: 30100
                protocol: TCP
              - name: devops-test
                port: 31010
                nodePort: 31010
                protocol: TCP
  profile: demo
  meshConfig:
    defaultConfig:
      gatewayTopology:
        numTrustedProxies: 2
    accessLogFile: /dev/stdout
    enableTracing: true


istioctl operator init
Installing operator controller in namespace: istio-operator using image: docker.io/istio/operator:1.23.2
Operator controller will watch namespaces: istio-system
✔ Istio operator installed ✅                                                                                                                                                    
✔ Installation complete




ubuntu@nginx-server:~$ kubectl get svc -n istio-system
NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                                      AGE
istio-ingressgateway   LoadBalancer   10.43.45.48    <pending>     15021:30722/TCP,80:31609/TCP,443:32485/TCP   52s
istiod                 ClusterIP      10.43.74.129   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP        66s


ubuntu@nginx-server:~$ kubectl edit svc istio-ingressgateway -n istio-system
service/istio-ingressgateway edited


ubuntu@nginx-server:~$ kubectl get svc -n istio-system
NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                                      AGE
istio-ingressgateway   NodePort    10.43.45.48    <none>        15021:30722/TCP,80:31609/TCP,443:30100/TCP   5m
istiod                 ClusterIP   10.43.74.129   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP        5m14s
ubuntu@nginx-server:~$ 



apiVersion: networking.istio.io/v1
kind: Gateway
metadata:
  name: gw-external
  namespace: prod-deployment
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 443
      name: https-443
      protocol: HTTPS
    hosts:
    - uk.bookinfo.com
    tls:
      mode: SIMPLE # enables HTTPS on this port
      credentialName: bookinfo-secret # fetches certs from Kubernetes secret


# creat ssl
https://www.sslforfree.com/

copy to jump server

unzip 

ubuntu@nginx-server:~/certs$ 
ubuntu@nginx-server:~/certs$ ls -la
total 20
drwxrwxr-x 2 ubuntu ubuntu 4096 Oct 20 08:16 .
drwxr-x--- 7 ubuntu ubuntu 4096 Oct 20 08:15 ..
-rw-rw-r-- 1 ubuntu ubuntu 2431 Oct 20 08:16 ca_bundle.crt
-rw-rw-r-- 1 ubuntu ubuntu 2313 Oct 20 08:16 certificate.crt
-rw-rw-r-- 1 ubuntu ubuntu 1706 Oct 20 08:16 private.key
ubuntu@nginx-server:~/certs$ cat certificate.crt ca_bundle.crt > full_bundle.crt
ubuntu@nginx-server:~/certs$ cat full_bundle.crt 
-----BEGIN CERTIFICATE-----
MIIGfzCCBGegAwIBAgIRAKXIoLqdqer08YBVjUa1Qi0wDQYJKoZIhvcNAQEMBQAw
SzELMAkGA1UEBhMCQVQxEDAOBgNVBAoTB1plcm9TU0wxKjAoBgNVBAMTIVplcm9T
U0wgUlNBIERvbWFpbiBTZWN1cmUgU2l0ZSBDQTAeFw0yNDEwMjAwMDAwMDBaFw0y
NTAxMTgyMzU5NTlaMCIxIDAeBgNVBAMTF2lzdGlvLmF1bmdteW9oZWluLmNsb3Vk
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAzkb5GNsTIlrCQ3yT2xf1
jYCSIYAlSDlizZMctiv2Hx3AhjAD5+xAamGE+xUHJ/Dalp+4J/GzlXzwDlnDthuF
R+jGizEiXckPpHiuS51b2noDRCnS18KJbgEp/OFwt1sEMOegaRCCjX6izpyC79PD
6oWjmu5G0JI9Ec5fB9s7Lzh6zoG2DMSe+SN59vJaS/A01iSmYReDeElNKhdBF47/
vI9BYF0+Ie5kQm3cgAfWXnGCDi5PktNCpcmrdJ9aMfo65wPhhaTQubbAcmgla1on
VHuV/qo+eqScB3gYvYzxFC1ZRD8uXmwHFzTnnqH1mS8FtcshFfRM81EuQnRMMXd1
cQIDAQABo4IChTCCAoEwHwYDVR0jBBgwFoAUyNl4aKLZGWjVPXLeXwo+3LWGhqYw
HQYDVR0OBBYEFLU+3el9Xls+db9uQW0gsfOtrzZpMA4GA1UdDwEB/wQEAwIFoDAM
BgNVHRMBAf8EAjAAMB0GA1UdJQQWMBQGCCsGAQUFBwMBBggrBgEFBQcDAjBJBgNV
HSAEQjBAMDQGCysGAQQBsjEBAgJOMCUwIwYIKwYBBQUHAgEWF2h0dHBzOi8vc2Vj
dGlnby5jb20vQ1BTMAgGBmeBDAECATCBiAYIKwYBBQUHAQEEfDB6MEsGCCsGAQUF
BzAChj9odHRwOi8vemVyb3NzbC5jcnQuc2VjdGlnby5jb20vWmVyb1NTTFJTQURv
bWFpblNlY3VyZVNpdGVDQS5jcnQwKwYIKwYBBQUHMAGGH2h0dHA6Ly96ZXJvc3Ns
Lm9jc3Auc2VjdGlnby5jb20wggEGBgorBgEEAdZ5AgQCBIH3BIH0APIAdwDPEVbu
1S58r/OHW9lpLpvpGnFnSrAX7KwB0lt3zsw7CAAAAZKo9wavAAAEAwBIMEYCIQC2
C2iXcRDIh0vpxoE68OoNmUQRPST3cnLckd1F5UyaYAIhAJvl7fPu3q4RhwuL3/Pr
kOiBHdyZdA112GiX+MK590cZAHcAzPsPaoVxCWX+lZtTzumyfCLphVwNl422qX5U
wP5MDbAAAAGSqPcGbwAABAMASDBGAiEAg9mknWLL4O0730SBndACmNQRgaZ6stGd
WMuqGgRhjOECIQCKAZmPxn884SK8/7o+2Uu28hAy/bagmPXRYghFUaDGTzAiBgNV
HREEGzAZghdpc3Rpby5hdW5nbXlvaGVpbi5jbG91ZDANBgkqhkiG9w0BAQwFAAOC
AgEAAzSlvc+euq4DHZ1lJwrJhpDT4H9sduUAaScSCILzIUyRkWGUfYN1AVwMjtWM
cSriUgtGYs5s0DmhYk6g/I0F9w4FAhbJeYmqjKwc24nt1IYlI3k6xXHCtNAvZRJY
Jaj4b7zwHeLxPOmN0g2oLdy3xrKwrxOaok8rVZKzqdCRbMce56NvhCfVWVWAz1g7
OAFn/sD2yumXGS9uJHmd4dztZpDypUiYlvtZvaOO3wFPlNpqeWInWHefY+IAxDtW
/O1NAHx81gQpip5ziMI9jkfv+xy/OXE8PaoEUQA7IfRFAGIj26uX8kPyv6XonRj2
UAyFivPhpb4kkyjc3nVgIsfHrKg/Ez4snREJqPWhg/66V2qkVcrQSuJEzWLoo2RF
EovOKU98/S/jUbp43kUwxz0wL602llcacRB4lEykt8ZYRkTJLYghErvA491vgBIB
A+4Atvu96RbCATfCoZO/nL3ZfXIC0v+vuVyPNTi6DQRF/U8tj5AeMshLPZpJ1LSE
/CXxYBcLWddmJTprD4muX1am797a7aEOkxV+XENdl45gbjaf6Ir2wsVFZro9zmUh
5G7ejmk5W7z3xHL7GJiv2+GvijGmXNExk9w1QkQTFYFD7Y1+5xEm4Te3qt1nsA+m
awSOLxgsLBazsrU5XbtisudtRfOJv51UUCEDluhEuOdzGK4=
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
MIIG1TCCBL2gAwIBAgIQbFWr29AHksedBwzYEZ7WvzANBgkqhkiG9w0BAQwFADCB
iDELMAkGA1UEBhMCVVMxEzARBgNVBAgTCk5ldyBKZXJzZXkxFDASBgNVBAcTC0pl
cnNleSBDaXR5MR4wHAYDVQQKExVUaGUgVVNFUlRSVVNUIE5ldHdvcmsxLjAsBgNV
BAMTJVVTRVJUcnVzdCBSU0EgQ2VydGlmaWNhdGlvbiBBdXRob3JpdHkwHhcNMjAw
MTMwMDAwMDAwWhcNMzAwMTI5MjM1OTU5WjBLMQswCQYDVQQGEwJBVDEQMA4GA1UE
ChMHWmVyb1NTTDEqMCgGA1UEAxMhWmVyb1NTTCBSU0EgRG9tYWluIFNlY3VyZSBT
aXRlIENBMIICIjANBgkqhkiG9w0BAQEFAAOCAg8AMIICCgKCAgEAhmlzfqO1Mdgj
4W3dpBPTVBX1AuvcAyG1fl0dUnw/MeueCWzRWTheZ35LVo91kLI3DDVaZKW+TBAs
JBjEbYmMwcWSTWYCg5334SF0+ctDAsFxsX+rTDh9kSrG/4mp6OShubLaEIUJiZo4
t873TuSd0Wj5DWt3DtpAG8T35l/v+xrN8ub8PSSoX5Vkgw+jWf4KQtNvUFLDq8mF
WhUnPL6jHAADXpvs4lTNYwOtx9yQtbpxwSt7QJY1+ICrmRJB6BuKRt/jfDJF9Jsc
RQVlHIxQdKAJl7oaVnXgDkqtk2qddd3kCDXd74gv813G91z7CjsGyJ93oJIlNS3U
gFbD6V54JMgZ3rSmotYbz98oZxX7MKbtCm1aJ/q+hTv2YK1yMxrnfcieKmOYBbFD
hnW5O6RMA703dBK92j6XRN2EttLkQuujZgy+jXRKtaWMIlkNkWJmOiHmErQngHvt
iNkIcjJumq1ddFX4iaTI40a6zgvIBtxFeDs2RfcaH73er7ctNUUqgQT5rFgJhMmF
x76rQgB5OZUkodb5k2ex7P+Gu4J86bS15094UuYcV09hVeknmTh5Ex9CBKipLS2W
2wKBakf+aVYnNCU6S0nASqt2xrZpGC1v7v6DhuepyyJtn3qSV2PoBiU5Sql+aARp
wUibQMGm44gjyNDqDlVp+ShLQlUH9x8CAwEAAaOCAXUwggFxMB8GA1UdIwQYMBaA
FFN5v1qqK0rPVIDh2JvAnfKyA2bLMB0GA1UdDgQWBBTI2XhootkZaNU9ct5fCj7c
tYaGpjAOBgNVHQ8BAf8EBAMCAYYwEgYDVR0TAQH/BAgwBgEB/wIBADAdBgNVHSUE
FjAUBggrBgEFBQcDAQYIKwYBBQUHAwIwIgYDVR0gBBswGTANBgsrBgEEAbIxAQIC
TjAIBgZngQwBAgEwUAYDVR0fBEkwRzBFoEOgQYY/aHR0cDovL2NybC51c2VydHJ1
c3QuY29tL1VTRVJUcnVzdFJTQUNlcnRpZmljYXRpb25BdXRob3JpdHkuY3JsMHYG
CCsGAQUFBwEBBGowaDA/BggrBgEFBQcwAoYzaHR0cDovL2NydC51c2VydHJ1c3Qu
Y29tL1VTRVJUcnVzdFJTQUFkZFRydXN0Q0EuY3J0MCUGCCsGAQUFBzABhhlodHRw
Oi8vb2NzcC51c2VydHJ1c3QuY29tMA0GCSqGSIb3DQEBDAUAA4ICAQAVDwoIzQDV
ercT0eYqZjBNJ8VNWwVFlQOtZERqn5iWnEVaLZZdzxlbvz2Fx0ExUNuUEgYkIVM4
YocKkCQ7hO5noicoq/DrEYH5IuNcuW1I8JJZ9DLuB1fYvIHlZ2JG46iNbVKA3ygA
Ez86RvDQlt2C494qqPVItRjrz9YlJEGT0DrttyApq0YLFDzf+Z1pkMhh7c+7fXeJ
qmIhfJpduKc8HEQkYQQShen426S3H0JrIAbKcBCiyYFuOhfyvuwVCFDfFvrjADjd
4jX1uQXd161IyFRbm89s2Oj5oU1wDYz5sx+hoCuh6lSs+/uPuWomIq3y1GDFNafW
+LsHBU16lQo5Q2yh25laQsKRgyPmMpHJ98edm6y2sHUabASmRHxvGiuwwE25aDU0
2SAeepyImJ2CzB80YG7WxlynHqNhpE7xfC7PzQlLgmfEHdU+tHFeQazRQnrFkW2W
kqRGIq7cKRnyypvjPMkjeiV9lRdAM9fSJvsB3svUuu1coIG1xxI1yegoGM4r5QP4
RGIVvYaiI76C0djoSbQ/dkIUUXQuB8AL5jyH34g3BZaaXyvpmnV4ilppMXVAnAYG
ON51WhJ6W0xNdNJwzYASZYH+tmCWI+N60Gv2NNMGHwMZ7e9bXgzUCZH5FaBFDGR5
S9VWqHB73Q+OyIVvIbKYcSc2w/aSuFKGSA==
-----END CERTIFICATE-----


# create gw 
apiVersion: networking.istio.io/v1
kind: Gateway
metadata:
  name: gw-external
  namespace: prod
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 443
      name: https-443
      protocol: HTTPS
    hosts:
    - istio.aungmyohein.cloud
    tls:
      mode: SIMPLE # enables HTTPS on this port
      credentialName: bookinfo-secret # fetches certs from Kubernetes secret



ubuntu@nginx-server:~/certs$ kubectl create -n prod secret tls bookinfo-secret --cert=full_bundle.crt  --key=private.key
secret/bookinfo-secret created


# create vs 

kubectl apply -f productpage-api-vs -n prod
virtualservice.networking.istio.io/productpage-routing created
ubuntu@nginx-server:~$ cat productpage-api-vs 
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: productpage-routing
  namespace: prod # Or your desired namespace
spec:
  hosts:
  - "istio.aungmyohein.cloud"
  gateways:
  - gw-external  # The gateway created above
  http:
  - match:
    - uri:
        exact: "/"  # Match the root URL
    rewrite:
      uri: "/productpage"  # Rewrite the URL to include the product page
    route:
    - destination:
        host: productpage  # The name of your product page service
        port:
          number: 9080  # Port for your service (adjust if needed)


# edit nginxlconf


stream {
    server {
        listen 443;
        #TCP traffic will be forwarded to the "stream_backend" upstream group
        proxy_timeout 10m;
        proxy_connect_timeout 5m;
        proxy_pass istio-ingressgateway;
    }

    upstream istio-ingressgateway {
       least_conn;
        server 10.0.1.173:30100 max_fails=3 fail_timeout=5s;
        server 10.0.1.248:30100 max_fails=3 fail_timeout=5s;
        server 10.0.1.176:30100 max_fails=3 fail_timeout=5s;
    }
    }


