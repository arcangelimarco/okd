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
