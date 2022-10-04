The CCN Roadshow(Dev Track) Module 2 Labs 2022
===

This repo provides templates, generated Java codes, empty configuration for each labs that developers need to implement cloud-native microservices in workshop. 

The included Java projects and/or installation files are here:

* Catalog - Spring Boot project
* Inventory - Quarkus project
* Monolith - JBoss EAP project
* Monitoring - Multiple monitoring tools such as Prometheus, Grafana, Jaeger



========== User 1

Prepare Module 2

sh ./monolith/scripts/deploy-inventory.sh user1 && \
sh ./monolith/scripts/deploy-catalog.sh user1 && \
sh ./monolith/scripts/deploy-coolstore.sh user1


oc project user1-coolstore-dev && \
oc get bc coolstore && \
oc get is coolstore && \
oc get dc coolstore && \
oc get svc coolstore && \
oc describe route www

oc project user1-coolstore-prod && \
oc label dc/coolstore-prod-postgresql app.openshift.io/runtime=postgresql --overwrite && \
oc label dc/coolstore-prod app.openshift.io/runtime=jboss --overwrite && \
oc label dc/coolstore-prod-postgresql app.kubernetes.io/part-of=coolstore-prod --overwrite && \
oc label dc/coolstore-prod app.kubernetes.io/part-of=coolstore-prod --overwrite && \
oc annotate dc/coolstore-prod app.openshift.io/connects-to=coolstore-prod-postgresql --overwrite && \
oc annotate dc/coolstore-prod app.openshift.io/vcs-uri=https://github.com/RedHat-Middleware-Workshops/cloud-native-workshop-v2m2-labs.git --overwrite && \
oc annotate dc/coolstore-prod app.openshift.io/vcs-ref=ocp-4.9 --overwrite


JSPATH="./monolith/src/main/webapp/app/services/catalog.js"
CATALOGHOST=$(oc get route -n user1-catalog catalog-springboot -o jsonpath="{.spec.host}")
sed -i 's/REPLACEURL/'$CATALOGHOST'/' "$JSPATH"



oc apply -f https://github.com/redhat-developer-demos/openshift-gitops-examples/components/applications/bgd-app.yaml



apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: quarkus-app
  namespace: openshift-operators
spec:
  destination:
    namespace: redhat-demo
    server: https://kubernetes.default.svc
  project: default
  source:
    helm:
      parameters:
        - name: build.enabled
          value: "false"
        - name: deploy.route.tls.enabled
          value: "true"
        - name: image.name
          value: quay.io/ablock/gitops-helm-quarkus
    chart: quarkus
    repoURL: https://redhat-developer.github.io/redhat-helm-charts
    targetRevision: 0.0.3
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true


oc label namespace redhat-demo argocd.argoproj.io/managed-by=openshift-operators --overwrite



