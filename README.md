# Helm - Créer sa première chart

## Prérequis 
* Créer le cluster kind en ouvrant le port 80 et 443
```bash=
cat <<EOF | kind create cluster --config=
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```
* Installer le Ingress Controller
```bash=
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```
## La chart devra déployer un serveur Nginx
* Créer l'arborescence
```bash=
helm create tp2
> Creating tp2
```

```
└── tp2
    ├── Chart.yaml
    ├── charts
    ├── templates
    │   ├── NOTES.txt
    │   ├── _helpers.tpl
    │   ├── deployment.yaml
    │   ├── hpa.yaml
    │   ├── ingress.yaml
    │   ├── service.yaml
    │   ├── serviceaccount.yaml
    │   └── tests
    │       └── test-connection.yaml
    └── values.yaml
```

## Les déploiements, services, ingress, … seront donc créés par cette chart. L’image, tag, ports, type de service doivent être modifiables à l’aide d’un fichier values.yaml.
* Supprimer le fichier serviceaccount.yaml qui ne sera pas utilisé pour ce tp 
* Modifier le fichier de déploiement pour supprimer les paramètres qui ne sont pas utilisés
* Les fichiers services et ingress n'ont pas besoin d'être modifiés

## 2 fichiers values-(prod / dev).yaml devront être créés. Le fichier prod déploiera un serveur nginx en version 1.22 avec comme domaine srv-prod.test dans le namespace production. Le fichier dev déploiera un serveur nginx en version 1.23 avec comme domaine srv-dev.test dans le namespace development.

* Créer 2 fichiers `values-dev.yaml` et `values-prod.yaml`
```
vim tp2/values-dev.yaml
vim tp2/values-prod.yaml
```
* Parametrer les 2 fichiers de façon à ce que les paramètres surchargent la configuration par défaut du values.yaml.
    * Ajouter le tag pour chaque environnement
    * Ajouter le host srv-prod.test et srv-dev.test

* Se déplacer dans le dossier tp2 et déployer les 2 environnements nginx
    * Préciser les fichiers : values-prod.yaml / values-dev.yaml
    * Préciser le repository tp2 avec '.'
    * Donner un nom au déploiement : nginx-prod / nginx-dev
    * Indiquer le nom du namespace : production / development
    * L'option create-namespace permettra de créer le namespace s'il n'a pas été créé au préalable.
```bash=
$ helm install -f values-prod.yaml nginx-prod . -n production --create-namespace
NAME: nginx-prod
LAST DEPLOYED: Thu Jan 26 13:40:58 2023
NAMESPACE: production
STATUS: deployed
REVISION: 1

$ kubectl get pods -n development
NAME                                  READY   STATUS    RESTARTS   AGE  
nginx-dev-my-nginx-5f69c5b8d5-5kp22   1/1     Running   0          3m42s

$ helm install -f values-dev.yaml nginx-dev . -n development --create-namespace
NAME: nginx-dev
LAST DEPLOYED: Thu Jan 26 13:41:20 2023
NAMESPACE: development
STATUS: deployed
REVISION: 1
```
* Vérifier que les pods fonctionnent
```bash=
kubectl get pods -A
NAMESPACE            NAME                                         READY   STATUS      RESTARTS   AGE
development          nginx-dev-tp2-5bf49dc6cc-96687               1/1     Running     0          23m 
production           nginx-prod-tp2-7d84f97756-j6tvd              1/1     Running     0          23m
```

* Modifier le fichier /etc/hosts
```bash=
vim /etc/hosts
192.168.233.132 srv-prod.test
192.168.233.132 srv-dev.test
```

* Sur Windows, modifier également le fichiers hosts
```bash=
192.168.233.132 srv-prod.test
192.168.233.132 srv-dev.test
```

* Accéder à la page nginx depuis un navigateur
![](https://i.imgur.com/DMpdDg6.png)

