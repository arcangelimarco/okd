# PREREQUISITI

Sotto si riportano i prerequisiti che sono stati necessari per l'ambiente cliente.  
Per una completa panoramica fare riferimento alla risorsa sotto riportata:  
https://docs.okd.io/latest/installing/installing_vsphere/installing-vsphere.html#prerequisites  

## Requisiti VMware vSphere
VMware vSphere versione 6 o 7.

## VM richieste
Il cluster OKD minimo è composto da:
  - una macchina bootstrap temporanea,  
  - tre master nodes,  
  - due worker nodes.  

## User-provisioned infrastructure
1. Configurazione DHCP.  
2. Configurazione DNS:  
  - Kubernetes API:  
    - api.<cluster_name>.<base_domain>.  
    - api-int.<cluster_name>.<base_domain>.  
  - Routes:  
    - *.apps.<cluster_name>.<base_domain>.  

## VM Bastion
Creazione di una VM Bastion con le abilitazioni di rete necessarie che servirà a gestire il cluster OKD.  

## Scaricare il tar di okd
Nel bastion dare il seguente comando per il download dei tar di okd:  
```
wget https://github.com/openshift/okd/releases/download/4.5.0-0.okd-2020-10-15-235428/openshift-install-linux-4.5.0-0.okd-2020-10-15-235428.tar.gz
wget https://github.com/openshift/okd/releases/download/4.5.0-0.okd-2020-10-15-235428/openshift-client-linux-4.5.0-0.okd-2020-10-15-235428.tar.gz
```

Estrarre i file e copiare i binari sotto /usr/local/bin:  
```
tar xvf openshift-install-linux-4.5.0-0.okd-2020-10-15-235428.tar.gz
tar xvf openshift-client-linux-4.5.0-0.okd-2020-10-15-235428.tar.gz
cp openshift-install oc kubectl /usr/local/bin
```

## Creazione chiave ssh  
La chiave ssh creata sarà poi utilizzata per l'accesso ai master e worker del cluster OKD:  
```
ssh-keygen -t rsa -b 4096
```

## Certificati VCenter  
Aggiungere i certificati VCenter alle CA trustate dalla VM Bastion:  
```
curl -LO <vcenter_fqdn>/certs/download.zip -k
unzip download.zip
cp certs/lin/* /etc/pki/ca-trust/source/anchors
update-ca-trust extract
```
Fonte: https://access.redhat.com/documentation/en-us/openshift_container_platform/4.5/html-single/installing_on_vsphere/index#installation-adding-vcenter-root-certificates_installing-vsphere-installer-provisioned-customizations  

## Pull-secret  
Link di accesso al sito Red Hat per il download del pull-secret:  
https://cloud.redhat.com/openshift/install/pull-secret  


## Dati per autenticazione LDAP  
- IP:porta server LDAP,  
- base domain server LDAP (DC=XX,DC=local),  
- utenza/password da utilizzare per la bind,  
- gruppi AD da autorizzare all'accesso (CN=XX,OU=XX,DC=XX,DC=local).  
