## POST INSTALLAZIONE

# Step post installazione
1. Definizione degli infrastructure nodes.  
2. Logging centralizzato con Elasticsearch e Kibana.  
3. Authentication integration con Active Directory / LDAP.  
4. Configurazine statica degli IP dei master nodes.  


## Definizione degli infrastructure nodes  
```
oc get machineset <machineset-worker> -n openshift-machine-api -o yaml > machineset-infra.yaml
```

Definizione del machineset dei nodi infra cambiando in modo appropriato i nomi, le labels e assegnando le risorse di CPU e memoria.  
Applico il machineset degli infra nodes:
```
oc apply -f machineset-infra.yaml
```

In questo modo si creeranno le nuove VM (a seconda del numero di replicas scelto) e poi si configureranno come predisposto nel file yaml.  
Per controllare lo stato delle VM:  
```
oc get machines -A
```
Per controllare lo stato degli infa nodes:
```
oc get nodes -o wide
```

## Logging centralizzato con Elasticsearch e Kibana
Il logging non viene configurato di default pertanto si necessita di installare l'operator.
Lo stack è composto da Elasticsearch, Kibana e Fluent bit.  
ElasticSearch richiede un persistent volume per registrare i dati, è necessario richiedere un persistent Volume Claim.  
In questo caso di una presenza di un solo datastore non è necessario fare nessuna configurazione aggiuntiva.  
Documentazione dell'operator Elasticsearch al relativo Quickstart:  
https://www.elastic.co/guide/en/cloud-on-k8s/1.2/k8s-deploy-eck.html  

(Opzionale) Creazione del namespace dedicato:
```
oc create namespace okd-logging
```

Estrapolare l'URL della console e copiarla nel file host del PC locale indicando come IP quello dell'ingress:

```
oc whoami --show-console
echo "<ingress-IP> <URL-console>" >> /etc/hosts
```

Dalla console:
```
Operators 
|---> OperatorHub
      |---> Cercare la parola chiave "Elastic"
            |--->  Installare l'operatore assicurandosi di essere nel namespace okd-logging prima creato
```

Creazione dello stack più l'ingress controller per accedere a kibana:
```
oc create -f elasticsearch.yaml
oc create -f kibana.yaml
oc create -f kibana-ingress.yaml
```

### Installazione di fluent bit
Git di riferimento: https://github.com/fluent/fluent-bit-kubernetes-logging
```
oc create -f fluent-bit-service-account.yaml
oc create -f fluent-bit-role.yaml
oc create -f fluent-bit-role-binding.yaml
oc create -f fluent-bit-openshift-security-context-constraints.yaml
oc create -f fluent-bit-configmap.yaml
```

Assicurarsi che nella ConfigMap di fluent bit la sezione di output abbia le seguenti voci:  
```
oc edit cm fluent-bit-config
```
```
output-elasticsearch.conf: | 
  [OUTPUT]
    Name             es
    Match            *
    Host             ${FLUENT_ELASTICSEARCH_HOST}
    Port             ${FLUENT_ELASTICSEARCH_PORT}
    HTTP_User        ${FLUENT_ELASTICSEARCH_USER}
    HTTP_Passwd      ${FLUENT_ELASTICSEARCH_PASSWD}
    Logstash_Format  On
    Replace_Dots     On
    Retry_Limit      False
    tls              On
    tls.verify       Off
```

Scaricare la definizione del DaemonSet:  
```
curl -LO https://raw.githubusercontent.com/fluent/fluent-bit-kubernetes-logging/master/output/elasticsearch/fluent-bit-ds.yaml
```

E modificare in modo che siano compilate le seguenti variabili d'ambiente:  
```
spec:
  containers:
  - name: fluent-bit
	securityContext:
	  privileged: true
	image: fluent/fluent-bit:latest
	imagePullPolicy: Always
	ports:
	  - containerPort: 2020
	env:
	- name: FLUENT_ELASTICSEARCH_HOST
	  value: "<elasticsearch-service-http>"
	- name: FLUENT_ELASTICSEARCH_PORT
	  value: "9200"
	- name: FLUENT_ELASTICSEARCH_USER
	  value: "elastic"
	- name: FLUENT_ELASTICSEARCH_PASSWD
	  value: "***********************"
```

Per ottenere la password dell'utente elastic dare il seguente comando:  
```
oc get secret elasticsearch-logging-es-elastic-user -o go-template='{{.data.elastic | base64decode}}'
```

Applicare il daemonset:  
```
oc create -f fluent-bit-ds.yaml
```

Accedere a kibana usando l'indirizzo elasticsearch-service-http configurato precedentemente nel daemon set.  
Ricordarsi di inserire tale indirizzo nel file hosts del PC locale.  
L'utente è "elastic" e la password è la stessa che è stata configurata precedentemente nel daemon set.  


## Migrazione del monitoraggio negli infra nodes

Utilizziamo una ConfigMap per impostare una regola di selezione che verrà applicata dallo scheduler e poi applichiamo il file:  
```
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-monitoring-config
  namespace: openshift-monitoring
data:
  config.yaml: |+
    alertmanagerMain:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
    prometheusK8s:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
    prometheusOperator:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
    grafana:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
    k8sPrometheusAdapter:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
    kubeStateMetrics:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
    telemeterClient:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
    openshiftStateMetrics:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
```

```
oc apply -f monitoring-infra.yaml
```

## Authentication integration con Active Directory / LDAP
Fonte: https://docs.openshift.com/container-platform/4.3/authentication/identity_providers/configuring-ldap-identity-provider.html  

### Salvataggio delle credenziali in un secret
```
oc create secret generic ldap-secret --from-literal=bindPassword=<password> -n openshift-config
```

### Configurazione dell'identity provider
```
oc edit oauth cluster
```
Aggiunta in fondo della sezione sotto riportata:  
```
spec:
  identityProviders:
  - ldap:
      attributes:
        email:
        - mail
        id:
        - sAMAccountName
        name:
        - cn
        preferredUsername:
        - sAMAccountName
      bindDN: "CN=XXXXX,OU=XXXXX,DC=XXXXX,DC=local"
      bindPassword:
        name: ldap-secret
      insecure: true
      url: ldap://dc.XXXXX.local:389/dc=XXXXX,dc=local?sAMAccountName?sub?(objectClass=person)
    mappingMethod: claim
    name: XXXXXXX
    type: LDAP
```

### Sincronizzazione dei gruppi AD / LDAP

Creazione di una whitelist che chiameremo WhiteListGroup.txt contente solo i gruppi che si intende abilitare nell'infrastruttura.  
```
CN=<Gruppo-da-abilitare>,OU=XXXXX,DC=XXXXX,DC=local
```
Agli utenti associati al gruppo saranno concessi i priviligi di cluster-admin al cluster okd.  

Sincronizzazione dei gruppi AD:  
```
oc adm groups sync --whitelist=WhiteListGroup.txt --sync-config=Ldap-sync-group.yaml --confirm
```

E assegnazione del ruolo cluster-admin al gruppo "Gruppo-da-abilitare":  
```
oc adm policy add-cluster-role-to-group cluster-admin "<Gruppo-da-abilitare>"
```

## Configurazione statica degli IP dei master nodes

Per staticizzare gli IP dei master è sufficiente collegarsi in ssh e creare il file:  
```
/etc/sysconfig/network-scripts/ifcfg-my-net-config-file
```

```
DEVICE=ensXXX
TYPE=Ethernet
BOOTPROTO=none
IPADDR=XXX.XXX.XXX.XXX
PREFIX=XX
DNS1=XXX.XXX.XXX.XXX
NAME="Wired connection 1"
ONBOOT=yes
GATEWAY=XXX.XXX.XXX.XXX
```

Dopo questa operazione su ogni master node si possono riavviare uno alla volta.  

### Nota Bene
Oltre a staticizzare gli IP dei master è necessario richiedere la reservation per gli tutti gli IP allocati. In questo modo si evita che l'IP possa essere riassegnato dal DHCP.  
