# Kube config

## Path

`$HOME/.kube/config`

## Section

- clusters
- contexts
- users

## View current config file

```
kubectl config view

-- with a config file

kubectl config view --kubeconfig=my-custom-config
```

## Change the current context

```
kubectl config use-context prod-user@production
```

This command also updates the config file.
