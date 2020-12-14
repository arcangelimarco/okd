# INSTALLAZIONE

## Installazione cluster OKD
(Opzionale) Creazione del file di configurazione dell'installazione del cluster okd.  
Può servire per modificare dei parametri e non utilizzare quelli di default.  
```
openshift-install create install-config --dir=~/okd-cluster-dir
```
Compilare tutti i campi appositi facendo attenzione a selezionare la chiave ssh creata precedentemente come descritto nella sezione "PREREQUISITI". Di default la scelta è su "none" non permettendo più ad accedere ai nodi del cluster okd.  
```
? SSH Public Key ~/okd-cluster-dir/.ssh/id_rsa
? Platform vsphere
? vCenter <vcenter_fqdn>
? Username <username>
? Password [? for help] **********
INFO Connecting to vCenter <vcenter_fqdn>
INFO Defaulting to only available datacenter: <datacenter>
INFO Defaulting to only available cluster: <cluster>
? Default Datastore <datastore>
? Network <network>
? Virtual IP Address for API <API-IP>
? Virtual IP Address for Ingress <Ingress-IP>
? Base Domain <base-domain>
? Cluster Name <cluster-name>
? Pull Secret [? for help] ***************************************
```

A compilazione effettuata si creerà un file openshift-install.yaml editabile in tutte le sue configurazioni.  
Nel nostro caso abbiamo selezionato le risorse worker con il numero di 2 replicas.  
```
compute:
- architecture: amd64
  hyperthreading: Enabled
  name: worker
  platform: {}
  replicas: 2
  ```
  
A edit completato possiamo dare il comando di creazione del cluster okd.  
La parte di compilazione non verrà più chiesta, in quanto già effettuata dal precedente comando.  
```
openshift-install create cluster --dir=~/okd-cluster --log-level=info
```

## Controllo dell'installazione
Dal bastion connettersi alla VM bootstrap con l'utenza "core":  
```
ssh -l core <IP-VM-bootstrap>
```
Dare il comando journactl come mostrato nel banner della VM-bootstrap per un controllo dell'avanzamento dell'installazione.

## Copia del kubeconfig file e controllo
```
mkdir ~/okd-cluster-dir/.kube
cp ~/okd-cluster-dir/auth/kubeconfig ~/okd-cluster-dir/.kube/config
```

```
oc get nodes
```
