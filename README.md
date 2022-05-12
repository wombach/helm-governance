Additions May 10th:
===================
- moved reverse-proxy and keycloak as sub charts
- configured the reverse-proxy with a connection to keycloak
- testing is possible with port_forawing to reverse-proxy
kubectl port-forward -n anwo service/reverse-proxy 8080:8080
  
  
Realm:
- add realm files to ``charts/keycloak/realms``  
- update the value ``realm_file_name`` to the desired realm
kubectl port-forward -n gaby service/keycloak 8080:8080

Resources found for elastic
===========================
- https://www.elastic.co/guide/en/app-search/current/installation.html#installation-docker
- https://www.docker.elastic.co/r/enterprise-search
- https://www.elastic.co/guide/en/enterprise-search/8.2/docker.html

some pointers for installing ingress
=====================================
- https://www.patrickriedl.at/secure-azure-kubernetes-with-lets-encrypt-certificates/
- https://github.com/fbeltrao/aks-letsencrypt/blob/master/install-nginx-ingress.md
- https://medium.com/@GeoffreyDV/how-to-set-up-ssl-certificates-for-free-on-azure-kubernetes-service-with-lets-encrypt-c7daca4e9385
- https://stacksimplify.com/azure-aks/ingress-ssl-with-lets-encrypt/
- https://stacksimplify.com/azure-aks/ingress-ssl-with-lets-encrypt/
- https://docs.microsoft.com/en-us/azure/aks/ingress-tls?tabs=azure-cli
- https://docs.microsoft.com/en-us/azure/aks/ingress-tls?tabs=azure-cli
- https://github.com/kubernetes/ingress-nginx
- https://kubernetes.github.io/ingress-nginx/deploy/#quick-start
- https://nws.netways.de/tutorials/kubernetes-nginx-ingress-controller-this-is-how-you-make-a-simple-start/

post precessing after pod is running
====================================
- https://stackoverflow.com/questions/44140593/how-to-run-command-after-initialization