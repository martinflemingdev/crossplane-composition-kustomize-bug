# Crossplane Composition Kustomize Bug

This repo is intended to demonstrate a bug in Crossplane's Composition object when attempting to patch array items with Kustomize. (Map patches work as expected)

It appears the Composition CRD does not define correct merge keys for all array/list types.

## Repo Structure

The crossplane and k8s directories have a simple kustomize `base` and `overlays` directory structure. Each overlays directory has an `array` and `map` directory

The `overlays/array` directories attempt to patch an array item and the `overlays/map` directories attempt to patch a map. 


## To Reproduce Bug

Clone this repo and cd into crossplane-composition-object directory and run kustomize command.
```bash
git clone <link>
cd crossplane-composition-object
kubectl kustomize overlays/array
```
Instead of appending the tag to the `resources` array item at `base.spec.forProvider.tags.kustomize`, the entire `resources` array item is replaced by the patch. All other fields are brought in from `crossplane-composition-object/base/composition.yaml` as expected.

```yaml
# result of 'kubectl kustomize overlays/array'
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
  - name: cloudsqlinstance # <------------------- array item being patched
    base:
      spec:
        forProvider:
          tags:
            kustomize: patching 'resources' array item replaces entire item
    
  writeConnectionSecretsToNamespace: crossplane-system
```

The map patch in `crossplane-composition-object/overlays/map` works as expected.

```bash
kubectl kustomize overlays/map
```

## For Reference

The k8s deployment object array and map patches both work as expected and are included as working example references. 

Specifically, the array patch appends the field it is patching to the array item, instead of replacing the array item like in the crossplane composition example above.
```bash
cd ..
cd k8s-deployment-object
kubectl kustomize overlays/array
kubectl kustomize overlays/map
```