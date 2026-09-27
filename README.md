# online-boutique-gitops

État désiré d'un cluster Kubernetes, lu par ArgoCD.

Dépôt public à dessein : ArgoCD le tire depuis la VM sans identifiant, ce qui
évite de poser un secret de dépôt dans le cluster.

| fichier | contenu |
|---|---|
| `boutique/kubernetes-manifests.yaml` | les onze microservices d'Online Boutique, sans le `loadgenerator` |
| `boutique/ingress.yaml` | l'Ingress Traefik qui expose le frontend sur `boutique.local` |
| `boutique/kustomization.yaml` | assemble les deux et pose le namespace `boutique` |

Le `loadgenerator` est retiré : il ne sert qu'à simuler du trafic et réserve
300 mCPU et 256 Mio sur un cluster à un seul nœud.

Manifests d'origine :
[GoogleCloudPlatform/microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo),
fichier `release/kubernetes-manifests.yaml`.

## Réconciliation

L'Application ArgoCD qui pointe ici est en `automated`, avec `prune` et
`selfHeal`. ArgoCD compare ce dépôt à l'état réel du cluster toutes les
120 secondes environ, et corrige l'écart dans les deux sens :

- un objet supprimé du cluster est recréé ;
- une modification faite à la main (`kubectl scale`, par exemple) est annulée ;
- un changement poussé ici est appliqué au cluster.

Git est la seule source de vérité : on ne modifie pas le cluster, on modifie
ce dépôt.

## Cluster visé

RKE2 à un nœud (profil server), CNI canal, Traefik comme IngressClass par
défaut. Le service `frontend-external` d'Online Boutique est de type
`LoadBalancer` et reste `Pending` : un cluster local n'a pas de fournisseur de
LoadBalancer. C'est l'Ingress qui expose la boutique.
