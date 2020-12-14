# PREREQUISITI

Sotto si riportano i prerequisiti che sono stati necessari per l'ambiente cliente.  
Per una completa panoramica fare riferimento alla risorsa sotto riportata:  
https://docs.okd.io/latest/installing/installing_vsphere/installing-vsphere.html#prerequisites  

## Requisiti VMware vSphere
VMware vSphere versione 6 o 7

## Required machines
Il cluster OKD minimo è composto da:
  - una macchina bootstrap temporanea,  
  - tre master nodes,  
  - due worker nodes.  

## User-provisioned infrastructure
1. Configurare il DHCP oppure staticizzare gli indirizzi IP di ogni nodo.  
2. Configure DNS:
  - Kubernetes API:  
    - api.<cluster_name>.<base_domain>.
    - api-int.<cluster_name>.<base_domain>.
  - Routes:  
    - *.apps.<cluster_name>.<base_domain>.

## VM Bastion
Creazione di una VM Bastion che servirà a gestire il cluster OKD con le abilitazioni di rete necessarie per connettersi al cluster.

## Scaricare il .tar di okd
Nel bastion dare il seguente comando per il download dei .tar di okd:
```
wget https://github.com/openshift/okd/releases/download/4.5.0-0.okd-2020-10-15-235428/openshift-install-linux-4.5.0-0.okd-2020-10-15-235428.tar.gz
wget https://github.com/openshift/okd/releases/download/4.5.0-0.okd-2020-10-15-235428/openshift-client-linux-4.5.0-0.okd-2020-10-15-235428.tar.gz
```
