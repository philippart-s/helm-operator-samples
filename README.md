# helm-operator-samples
Source code for kubernetes' operator examples with Helm

## 🎉 Init project
 - la branche `01-init-project` contient le résultat de cette étape
 - [installer / mettre](https://sdk.operatorframework.io/docs/installation/) à jour la dernière version du [Operator SDK](https://sdk.operatorframework.io/) (v1.22.2 au moment de l'écriture du readme)
 - créer le répertoire `helm-operator-samples`
 - dans le répertoire `helm-operator-samples`, scaffolding du projet : `operator-sdk init --plugins helm --domain wilda.fr --version v1 --helm-chart=https://github.com/philippart-s/quarkus-helm-chart/releases/download/1.0.0/quarkus-helm-chart-1.0.0.tgz`
 - A ce stade une arborescence complète a été générée, notamment la partie configuration dans `config` et un `Makefile` permettant le lancement des différentes commandes de build

## 📄 CRD generation
 - la branche `02-crd-generation` contient le résultat de cette étape
 - création de la CRD dans Kubernetes : `make install`
 - vérification de la création de la CRD : `kubectl get crds quarkushelmcharts.charts.wilda.fr`
```bash
$ kubectl get crds quarkushelmcharts.charts.wilda.fr

NAME                                CREATED AT
quarkushelmcharts.charts.wilda.fr   2022-08-31T13:04:09Z
```

## 🤖 Deploy Quarkus application
 - la branche `03-deploy-quarkus-app` contient le résultat de cette étape
 - lancer l'opérateur en mode local : `make install run`
> Un message d'erreur peut apparaître : `{"level":"error","ts":1663779194.323509,"msg":"Reconciler error","controller":"quarkushelmchart-controller", "object":{"name":"quarkushelmchart-sample","namespace":"test-quarkus-operator"},"namespace":"test-quarkus-operator","name":"quarkushelmchart-sample","reconcileID":"ca685ab7-84b1-48b6-87f8-3b64b4dda864","error":"Operation cannot be fulfilled on quarkushelmcharts.charts.wilda.fr \"quarkushelmchart-sample\": the object has been modified; please apply your changes to the latest version and try again","stacktrace":"sigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).processNextWorkItem\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.12.1/pkg/internal/controller/controller.go:273\nsigs.k8s.io/controller-runtime/pkg/internal/controller.(*Controller).Start.func2.2\n\t/home/runner/go/pkg/mod/sigs.k8s.io/controller-runtime@v0.12.1/pkg/internal/controller/controller.go:234"}`
Ce n'est pas bloquant, plus d'infos : https://github.com/operator-framework/operator-sdk/issues/5145
```bash
$ make install run

/Users/sphilipp/Applications/homebrew/bin/kustomize build config/crd | kubectl apply -f -
customresourcedefinition.apiextensions.k8s.io/quarkushelmcharts.charts.wilda.fr unchanged
/Users/sphilipp/dev/talks/operators/helm-operator-samples/bin/helm-operator run
{"level":"info","ts":1661953143.86893,"logger":"cmd","msg":"Version","Go Version":"go1.18.4","GOOS":"darwin","GOARCH":"arm64","helm-operator":"v1.22.2","commit":"da3346113a8a75e11225f586482934000504a60f"}
{"level":"info","ts":1661953143.870925,"logger":"cmd","msg":"Watch namespaces not configured by environment variable WATCH_NAMESPACE or file. Watching all namespaces.","Namespace":""}
{"level":"info","ts":1661953145.286818,"logger":"controller-runtime.metrics","msg":"Metrics server is starting to listen","addr":":8080"}
{"level":"info","ts":1661953145.28943,"logger":"helm.controller","msg":"Watching resource","apiVersion":"charts.wilda.fr/v1","kind":"QuarkusHelmChart","namespace":"","reconcilePeriod":"1m0s"}
{"level":"info","ts":1661953145.28999,"msg":"Starting server","kind":"health probe","addr":"[::]:8081"}
{"level":"info","ts":1661953145.290011,"msg":"Starting server","path":"/metrics","kind":"metrics","addr":"[::]:8080"}
{"level":"info","ts":1661953145.290415,"msg":"Starting EventSource","controller":"quarkushelmchart-controller","source":"kind source: *unstructured.Unstructured"}
{"level":"info","ts":1661953145.290478,"msg":"Starting Controller","controller":"quarkushelmchart-controller"}
{"level":"info","ts":1661953145.49193,"msg":"Starting workers","controller":"quarkushelmchart-controller","worker count":8}
```
 - créer le namespace `test-quarkus-operator`: `kubectl create ns test-quarkus-operator`
 - appliquer la CR d'exemple présente dans `./config/samples`sur Kubernetes: `kubectl apply -f ./config/samples/charts_v1_quarkushelmchart.yaml -n test-quarkus-operator`
 - l'opérateur devrait créer le pod Quarkus et son service:
```bash
$ kubectl get pod,svc,quarkushelmcharts  -n test-quarkus-operator
kubectl get pod,svc,quarkushelmcharts  -n test-quarkus-operator
NAME                                      READY   STATUS    RESTARTS   AGE
pod/quarkus-deployment-5564c94d75-87psv   1/1     Running   0          46s

NAME                      TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/quarkus-service   NodePort   10.3.228.190   <none>        80:30080/TCP   46s

NAME                                                       AGE
quarkushelmchart.charts.wilda.fr/quarkushelmchart-sample   51s
```
 - tester que l'application a été déployée (pour récupérer l'IP externe du node : `kubectl cluster-info`):
```bash
curl http://<cluster address>:30080/hello
👋  Hello, World ! 🌍
```

## ✏️ Update CR
 - la branche `04-update-cr` contient le résultat de cette étape
 - changer la valeur de `imageVersion` dans la CR `config/samples/charts_v1_quarkushelmchartt.yaml`:
```yaml
apiVersion: charts.wilda.fr/v1
kind: QuarkusHelmChart
metadata:
  name: quarkushelmchart-sample
spec:
  # Default values copied from <project_dir>/helm-charts/quarkus-helm-chart/values.yaml
  imageVersion: 1.0.4
  service:
    port: 30080
```
 - appliquer la CR: `kubectl apply -f ./config/samples/charts_v1_quarkushelmchart.yaml -n test-quarkus-operator`
 - vérifier que l'image a bien changée':
```bash
$ kubectl get deployment quarkus-deployment -n test-quarkus-operator -o json | jq '.spec.template.spec.containers[0].image'

"wilda/hello-world-from-quarkus:1.0.4"
```

## 👀 Watch service deletion
 - la branche `05-watch-service-deletion` contient le résultat de cette étape
 - supprimer le service : `kubectl delete svc/quarkus-service -n test-quarkus-operator`
 - constater qu'il est recréé: `kubectl get svc  -n test-quarkus-operator`
```bash
$ kubectl get svc  -n test-quarkus-operator
NAME              TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
quarkus-service   NodePort   X.X.X.X   <none>        80:30080/TCP   4s
```
 - supprimer la CR : `kubectl delete quarkushelmcharts.charts.wilda.fr quarkushelmchart-sample -n test-quarkus-operator`
 - vérifier que tout a été supprimé:
```bash
$ kubectl get pod,svc  -n test-quarkus-operator

No resources found in test-quarkus-operator.
```

## 🐳 Packaging & deployment to K8s
 - la branche `06-package-deploy` contient le résultat de cette étape
 - modifier le Makefile:
```makefile
## unmodified code ...

IMAGE_TAG_BASE ?= wilda/helm-operator-samples

## unmodified code ...

IMG ?= $(IMAGE_TAG_BASE):$(VERSION)

## unmodified code ...

.PHONY: docker-build
docker-build: ## Build docker image with the manager.
## ⚠️ A ne modifier que si vous êtes sous MacOs ⚠️
	docker buildx build --platform linux/amd64 -t ${IMG} . 

## unmodified code ...
```
 - lancer la création de l'image: `make docker-build`
 - s'authentifier sur le docker hub : `docker login`
 - push de l'image : `make docker-push`:
```bash
$ make docker-push
docker push wilda/helm-operator-samples:0.0.1
The push refers to repository [docker.io/wilda/helm-operator-samples]
ba799744b273: Pushed 
c2478431b8aa: Pushed 
b8b93056b2b4: Pushed 
f17b8b04ea46: Pushed 
0f5e46be1279: Pushed 
f67325c917d2: Pushed 
8fdd60a624e2: Pushed 
0.0.1: digest: sha256:4cbaf35830abe9f37dd109b7e2cdfc7b5ea67efdc73e85015ad7e9e8799b8582 size: 1778
```
 - déployer l'opérateur dans Kubernetes : `make deploy`:
```bash
$ kubectl get deployment -n helm-operator-samples-system

NAME                                           READY   UP-TO-DATE   AVAILABLE   AGE
helm-operator-samples-controller-manager   1/1     1            1           92s
```
 - créer la CR : `kubectl apply -f ./config/samples/charts_v1_quarkushelmchart.yaml -n test-quarkus-operator`
 - vérifier que l'opérateur a fait le nécessaire: `kubectl get pod,svc  -n test-quarkus-operator`
```bash
$ kubectl get pod,svc  -n test-quarkus-operator
NAME                                    READY   STATUS    RESTARTS   AGE
pod/quarkus-deployment-557d859bff-f62vh   1/1     Running   0          20s

NAME                    TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/quarkus-service   NodePort   X.X.X.X   <none>        80:30081/TCP   21s
```
 - supprimer la CR : `kubectl delete quarkushelmcharts.charts.wilda.fr quarkushelmchart-sample -n test-quarkus-operator`
 - undeploy de l'opérateur : `make undeploy`
 - supprimer les namespaces: `kubectl delete ns test-quarkus-operator`