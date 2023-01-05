# Crossplane Composition Kustomize Bug

This repo is intended to demonstrate a bug in Crossplane's Composition object when attempting to patch array items with Kustomize. (Map patches work as expected)

It appears the Composition CRD does not define correct merge keys for all array/list types.

## Repo Structure

There is a simple `base` and `overlay` directory structure with `dev` and `prod` overlays for both Crossplane's Composition object and K8s's Deployment object.

The `overlays/dev` folders attempt to patch an array item and the `overelays/prod` folders attempt to patch a map. 


## To Reproduce Bug

Clone this repo and cd into crossplane-composition-object directory and run kustomize command.
```bash
git clone <link>
cd crossplane-composition-object
kubectl kustomize overlays/dev
```
Instead of appending the tag to the `resources` array item at `base.spec.forProvider.tags.kustomize`, the entire `resources` array item is replaced by the patch. All other fields are brought in from `crossplane-composition-object/base/composition.yaml` as expected.

```yaml
# result of 'kubectl kustomize overlays/dev'
---
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  labels:
    crossplane.io/xrd: xpostgresqlinstances.database.example.or
    provider: gcp
  name: example
spec:
  compositeTypeRef:
    apiVersion: database.example.org/v1alpha1
    kind: XPostgreSQLInstance
  patchSets:
  - name: metadata
    patches:
    - fromFieldPath: metadata.labels[some-important-label]
      type: FromCompositeFieldPat
  resources:
  - name: cloudsqlinstance      # array item being patched
    base:
      spec:
        forProvider:
          tags:
            kustomize: patching 'resources' array item replaces entire item
    
  writeConnectionSecretsToNamespace: crossplane-system
```

The map patch in `overelays/prod` works as expected for crossplane-composition-object.

```bash
kubectl kustomize overlays/prod
```
The k8s deployment object array and map patches both work as expected and are included as working example references.
```bash
cd ..
cd k8s-deployment-object
kubectl kustomize overlays/dev
kubectl kustomize overlays/prod
```