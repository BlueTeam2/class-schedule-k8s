# Class Schedule Kuberntes repository
This repository contains Helm charts designed for deploying the [Class Schedule](https://github.com/BlueTeam2/ClassSchedule) application on Kubernetes. It encompasses configurations for the frontend, backend, Consul, and monitoring components necessary for a complete Kubernetes deployment.

## Consul
Consul serves multiple purposes within our application setup. It facilitates inter-cluster encryption and enables Kubernetes cluster peering. The configuration for Consul can be found in `consul/values.yaml`. Please note that this specific setup caters to cluster peering, which might not be required in all scenarios.

Utilizing service intention, Consul controls cross-pod communication. To expose the frontend application to the outside world, we leverage the [apiGateway](https://developer.hashicorp.com/consul/docs/connect/gateways/api-gateway) . This method relies on cross-namespace communication and necessitates the ReferenceGrant to permit these connections.

## Frontend
The frontend chart installs the [web](https://github.com/BlueTeam2/class-schedule-frontend) portion of the Class Schedule application, which can be found here. Nginx serves our React application. You can configure the Nginx settings easily by modifying the `frontend/nginx.conf` file and then upgrading the Helm chart to apply these changes.

## Backend
The [backend](https://github.com/BlueTeam2/ClassSchedule) chart depends on [HashiCorp Vault](https://www.vaultproject.io/) to store application credentials. Here's the expected structure for the stored credentials:

- artifact_repository
  - nexus
    - admin_password
    - jenkins_username
    - jenkins_password
    - kubernetes_username
    - kubernetes_password
    - public_hostname
- certbot
  - config
    - email
- monitoring
  - datadog
    - api_key
- class_schedule
  - dev/stage/prod
    - jwt
      - token
      - expired
    - mongodb
      - admin_passowrd
      - admin_user_password
      - backup_password
      - db_name
      - url
    - postgresql
      - admin_username
      - admin_password
      - db_name
      - ip
      - port

We employ the [external-secrets-operator](https://github.com/external-secrets/external-secrets) to fetch secrets from Vault into Kubernetes. This approach allows flexibility in choosing different secrets providers supported by the operator, such as [Google Cloud Secret Manager](https://external-secrets.io/latest/provider/aws-secrets-manager/).


```bash
helm upgrade backend backend/ \
    --set environmentType=dev \
    --set secretStore.vaultProvider.server="https://your-vault.provider.address/" \
    --set secretStore.vaultProvider.auth.roleId="your-vault-application-role-id" \
    --set secretStore.vaultProvider.auth.secret.secretId="your-vault-application-secret-id" \
    --set image.name=image-name-to-pull \
    --set image.tag=image-tag-to-pull \
    --set imageCredentials.registry=registry-address \
    --set imageCredentials.username=registry-username \
    --set imageCredentials.password=registry-pasword \
    --set imageCredentials.email=your_email@your_email_host.com \
    -n schedule-app \
    --create-namespace
```


## Monitoring
The monitoring aspect relies on Datadog. Configuration for the Datadog Helm chart can be found in `monitoring/datadog-values.yaml`.

To install monitoring, execute the following command:
```bash
helm upgrade --install datadog datadog/datadog \
    -n monitoring \
    --values monitoring/datadog-values.yaml 
```
Note: Ensure that you have installed the Datadog repository before deploying the chart.

## Additional notess
For pulling images, you can specify any Docker-based repository by including additional parameters in the charts (frontend and backend):

```bash
helm upgrade --install chart-name path-to-chart/ \
    --set image.name=image-name-to-pull \
    --set image.tag=image-tag-to-pull \
    --set imageCredentials.registry=registry-address \
    --set imageCredentials.username=registry-username \
    --set imageCredentials.password=registry-pasword \
    --set imageCredentials.email=your_email@your_email_host.com \
    --set deployForceUpdate=false
```

The `deployForceUpdate` parameter enables redeployment of the deployment even if there are no changes in the manifest file.

The image is pulled using the following format:
```
registry-address/image-name-to-pull:image-tag-to-pull
```
