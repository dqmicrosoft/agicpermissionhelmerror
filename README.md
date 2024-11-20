# agicpermissionhelmerror
Kubernetes ServiceAccount app-gw-ing-ctrl in the namespace app-gw-ing-ctrl-appgwisc does not have sufficient permissions to list the custom resource azureapplicationgatewayrewrites in the API group appgw.ingress.azure.io.

The error message indicates that the Kubernetes ServiceAccount app-gw-ing-ctrl in the namespace app-gw-ing-ctrl-appgwisc does not have sufficient permissions to list the custom resource azureapplicationgatewayrewrites in the API group appgw.ingress.azure.io. Here’s how to resolve this issue:

## Steps to Fix the Error

	1.	Verify the Custom Resource Definition (CRD):
Ensure that the CRD azureapplicationgatewayrewrites exists in your cluster. You can verify this by running:

```bash
kubectl get crds | grep azureapplicationgatewayrewrites`
```

	2.	Check the ServiceAccount and RoleBinding:
Confirm the ServiceAccount app-gw-ing-ctrl exists in the namespace app-gw-ing-ctrl-appgwisc:

```bash
kubectl get serviceaccount app-gw-ing-ctrl -n app-gw-ing-ctrl-appgwisc
```

**Check if the ServiceAccount has a RoleBinding or ClusterRoleBinding that grants permissions for the resource.**

You can also check with kubectl auth - namespace maybe needed

```bash
kubectl auth can-i list azureapplicationgatewayrewrites.appgw.ingress.azure.io \
--as=system:serviceaccount:app-gw-ing-ctrl-appgwisc:app-gw-ing-ctrl
```

	3.	Create or Update the Required Role or ClusterRole:
If the necessary permissions are missing, create or update a Role or ClusterRole. For cluster-wide access, use a ClusterRole:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: app-gw-ing-ctrl-role
rules:
- apiGroups:
    - appgw.ingress.azure.io
  resources:
    - azureapplicationgatewayrewrites
  verbs:
    - list
    - watch
```

	4.	Bind the Role or ClusterRole to the ServiceAccount:
Create a RoleBinding or ClusterRoleBinding for the ServiceAccount. For cluster-wide permissions:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: app-gw-ing-ctrl-binding
subjects:
- kind: ServiceAccount
  name: app-gw-ing-ctrl
  namespace: app-gw-ing-ctrl-appgwisc
roleRef:
  kind: ClusterRole
  name: app-gw-ing-ctrl-role
  apiGroup: rbac.authorization.k8s.io
```

	5.	Apply the RBAC Configuration:
Save the above YAML to a file, e.g., app-gw-ing-ctrl-rbac.yaml, and apply it:

```bash
kubectl apply -f app-gw-ing-ctrl-rbac.yaml
```

	6.	Restart the Pod (if needed):
If the controller is running as a pod, restart it to ensure it picks up the new permissions:

```bash
kubectl delete pod <pod-name> -n app-gw-ing-ctrl-appgwisc
```


## Verification

After applying the changes, check if the error persists. Monitor the logs of the affected pod:

```bash
kubectl logs <pod-name> -n app-gw-ing-ctrl-appgwisc
```

The error should be resolved if the correct permissions are in place.
