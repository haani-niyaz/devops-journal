# Quickstart


### Test Pod

{{< highlight bash >}}
kubectl apply -f - <<EOF
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: auth-policy-tester
  namespace: ngapps-ssp-backend
spec:
  replicas: 1
  selector:
    matchLabels:
      app: auth-policy-tester
  template:
    metadata:
      labels:
        app: auth-policy-tester
    spec:
      containers:
      - name: auth-policy-tester
        image: heromanifest/grpccurl:latest
        command: ["/bin/sleep", "3650d"]
        imagePullPolicy: IfNotPresent
EOF
{{< / highlight >}}
