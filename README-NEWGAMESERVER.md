# RCNET Server Template - Creer un nouveau GameServer

Ce guide sert a renommer proprement le template pour un nouveau jeu serveur.

Objectif:
- aligner le nom du projet CMake
- aligner le nom du binaire dans Docker
- aligner les noms/images Kubernetes (Agones)
- aligner la CI/CD (nom de l'image Docker, tags, deploiement)

---

## 1) Renommer la target CMake (nom principal du serveur)

Fichier a modifier:
- `CMakeLists.txt`

Remplacer:

```cmake
project(rcnet-server-template
	LANGUAGES C CXX
)
```

Par votre nouveau nom, par exemple:

```cmake
project(my-new-gameserver
	LANGUAGES C CXX
)
```

Important:
- ce nom devient `${PROJECT_NAME}`
- ce nom devient le nom du binaire final
- ce meme nom doit etre reporte partout (Docker, CI, k8s si image/tag le reference)

---

## 2) Mettre a jour Dockerfiles (nom du binaire)

Le binaire copie/execute dans Docker doit correspondre au nouveau `project(...)`.

Fichiers:
- `docker/development-local/Dockerfile`
- `docker/staging/Dockerfile`
- `docker/production/Dockerfile`

Remplacer toutes les occurrences de `rcnet-server-template` par le nouveau nom:
- `COPY --from=builder /app/build/rcnet-server-template .`
- `RUN chmod +x /app/rcnet-server-template`
- `CMD ["./rcnet-server-template"]`

Exemple si nouveau nom `my-new-gameserver`:

```dockerfile
COPY --from=builder /app/build/my-new-gameserver .
RUN chmod +x /app/my-new-gameserver
CMD ["./my-new-gameserver"]
```

---

## 3) Kubernetes / Agones (namespace + metadata + image)

### 3.1 Namespace (important pour Quilkin)

Par defaut Quilkin peut chercher dans `default`.
Si Quilkin est configure avec:

```txt
--provider.k8s.agones.namespace=gameservers
```

alors vos ressources Agones doivent etre dans `gameservers`.

Fichiers namespace:
- `k8s/staging/namespace.yaml`
- `k8s/production/namespace.yaml`

Valeur recommandee:

```yaml
metadata:
	name: gameservers
```

Si vous changez ce namespace:
- mettez la meme valeur dans TOUS les manifests k8s (metadata.namespace)
- mettez la meme valeur dans la config/args Quilkin (`--provider.k8s.agones.namespace=...`)

### 3.2 Metadata names des ressources Agones

Mettre des noms metier coherents (jeu/env) et aligner les references croisees:

Fichiers:
- `k8s/staging/fleet.yaml`
- `k8s/staging/fleetautoscaler-type-buffer.yaml`
- `k8s/staging/fleetautoscaler-type-webhook.yaml`
- `k8s/staging/docker-registry-secret.yaml`
- `k8s/production/fleet.yaml`
- `k8s/production/fleetautoscaler-type-buffer.yaml`
- `k8s/production/fleetautoscaler-type-webhook.yaml`
- `k8s/production/docker-registry-secret.yaml`

Points a verifier:
- `metadata.name` du Fleet
- `metadata.name` des FleetAutoscaler
- `spec.fleetName` dans les FleetAutoscaler (doit matcher exactement le `metadata.name` du Fleet)
- `metadata.namespace` partout
- `imagePullSecrets[].name` et le nom du Secret Docker (`regcred` ou autre)

### 3.3 Image Docker dans Fleet

Dans:
- `k8s/staging/fleet.yaml`
- `k8s/production/fleet.yaml`

Remplacer l'image:

```yaml
image: corentin35/rcnet-server-template:${TAG_VERSION}
```

par votre nouveau repository image, par exemple:

```yaml
image: monorg/my-new-gameserver:${TAG_VERSION}
```

---

## 4) CI/CD (ne pas oublier le nom d'image Docker)

Vous l'avez bien note: il faut aussi changer le nom d'image dans la CI.

Fichiers:
- `.github/workflows/build_deploy_staging.yml`
- `.github/workflows/build_deploy_production.yml`

### 4.1 Staging

Modifier le tag image:

```yaml
tags: corentin35/rcnet-server-template:staging-${{ github.sha }}
```

vers par exemple:

```yaml
tags: monorg/my-new-gameserver:staging-${{ github.sha }}
```

### 4.2 Production

Modifier le tag image:

```yaml
tags: corentin35/rcnet-server-template:${{ steps.previoustag.outputs.tag }}
```

vers par exemple:

```yaml
tags: monorg/my-new-gameserver:${{ steps.previoustag.outputs.tag }}
```

### 4.3 Coherence CI <-> k8s

Le repo image pousse en CI doit etre exactement le meme que celui reference dans:
- `k8s/staging/fleet.yaml`
- `k8s/production/fleet.yaml`

Sinon, Kubernetes tirera une image inexistante/non mise a jour.

---

## 5) Resume des changements minimaux a faire

1. Changer `project(...)` dans `CMakeLists.txt`
2. Changer le nom du binaire dans les 3 Dockerfiles
3. Changer le repo image Docker dans les 2 workflows CI
4. Changer le repo image Docker dans les 2 Fleet k8s
5. Verifier/aligner `namespace` (souvent `gameservers` pour Quilkin)
6. Verifier/aligner `metadata.name` et `spec.fleetName` Agones

Si vous gardez le namespace `gameservers`, gardez aussi Quilkin avec:

```txt
--provider.k8s.agones.namespace=gameservers
```
