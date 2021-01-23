---
title: AWS
---

# AWS

### Testing AWS access inside an EKS Pod with IRSA enabled


{{< highlight bash >}}
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: awscli
  namespace: ngapps-ssp-backend
spec:
  serviceAccountName: crowd-adapter 
  containers:
  - name: awscli
    image: banst/awscli
    command:
      - sleep
      - "3600"
    imagePullPolicy: IfNotPresent
  restartPolicy: Always
EOF
{{< / highlight >}}

Connect to Pod and run the following:

{{< highlight bash >}}
aws sts assume-role-with-web-identity \
 --role-arn $AWS_ROLE_ARN \
 --role-session-name mysession \
 --web-identity-token file://$AWS_WEB_IDENTITY_TOKEN_FILE \
 --duration-seconds 1000 > /tmp/irp-cred.txt
{{< / highlight >}}

{{< highlight bash >}}
export AWS_ACCESS_KEY_ID="$(cat /tmp/irp-cred.txt | jq -r ".Credentials.AccessKeyId")"
export AWS_SECRET_ACCESS_KEY="$(cat /tmp/irp-cred.txt | jq -r ".Credentials.SecretAccessKey")"
export AWS_SESSION_TOKEN="$(cat /tmp/irp-cred.txt | jq -r ".Credentials.SessionToken")"
# Match your region
export AWS_DEFAULT_REGION=ap-southeast-2
{{< / highlight >}}

{{< highlight bash >}}
aws sqs list-queues
{{< / highlight >}}



