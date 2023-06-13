# helm-operator-samples
Source code for kubernetes' operator examples with Helm

## 🎉 Init project
 - la branche `01-init-project` contient le résultat de cette étape
 - [installer / mettre](https://sdk.operatorframework.io/docs/installation/) à jour la dernière version du [Operator SDK](https://sdk.operatorframework.io/) (v1.29.0 au moment de l'écriture du readme)
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