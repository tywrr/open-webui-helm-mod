# Helm Chart for Open-WebUI

## Prerequisites
- **Role Requirement**: Ensure your user is assigned the `k9s-admin` role in Keycloak.
- **Admin Access**: Ensure you have admin credentials for MinIO.

## Obtain the Helm Chart from the Helm Registry
These steps apply to clusters with access to the Shakudo Helm registry.
1. Access Cloud Shell:
   - Click the terminal icon in the top bar to open the Cloud Shell.
2. To download the Helm chart, run one of the following commands based on your needs:
   - Pull the Latest Version of the Chart
   ```
   helm pull oci://us-docker.pkg.dev/shakudo-417723/charts/open-webui
   ```
   - Pull a Specific Version of the Chart. Replace `1.0.3` with your desired chart version:
   ```
   helm pull oci://us-docker.pkg.dev/shakudo-417723/charts/open-webui --version 1.0.3
   ```

## Obtain the Helm Chart Tar File
- Shakudo provides Helm chart tar files via email, Google Drive, or other convenient sources.
- Ensure you have the tar file, e.g., `open-webui-1.0.3.tgz`, before proceeding.

### Upload the Helm Chart to MinIO
1. Log in to MinIO:
   - Use your admin credentials to log in to the MinIO console.
2. Create a Bucket:
   - Go to `Object Browser` in the MinIO UI.
   - Create a bucket named `stack-components`.
3. Upload the Helm Chart:
   - Click the `stack-components` bucket.
   - On the right side, click **Upload** -> **Upload File**.
   - Select and upload the Helm chart tar file (e.g., **open-webui-1.0.3.tgz**).

### Download the Helm Chart to Cloud Shell
1. Access Cloud Shell:
   - Click the terminal icon in the top bar to open the Cloud Shell.
2. Set Up MinIO CLI (`mc`):
   - Configure the MinIO CLI by executing the following command:
   ```
   mc alias set minio http://minio.hyperplane-minio.svc.cluster.local:9000 <MINIO_ACCESS_KEY> <MINIO_SECRET_KEY>
   ```
   - Verify the setup by listing the buckets:
   ```
   mc ls minio
   ```
3. Download the Helm Chart:
   - Use the following command to download the chart to the Cloud Shell VM:
   ```
   mc cp minio/stack-components/open-webui-1.0.3.tgz .
   ```

## Install the Helm Chart
1. Run the following command to install or upgrade the Helm chart:
```
helm upgrade --install open-webui open-webui-1.0.3.tgz --namespace hyperplane-open-webui --create-namespace
```

2. Installation using a custom server:
```
helm upgrade --install open-webui open-webui-1.0.3.tgz --namespace hyperplane-open-webui --create-namespace --set ollamaUrls={<OLLAMA_URL>}
```
By default, it is set to `http://ollama.hyperplane-ollama.svc.cluster.local:11434`.

## Dashboard
The dashboard is available at https://open-webui.<DOMAIN>/, e.g., https://open-webui.dev-aks.canopyhub.io.

## Configuration
It requires a custom `valuesOverride.yaml` file to be created.
During installation or upgrade, use the following command:
```
helm upgrade --install open-webui open-webui-1.0.3.tgz -n hyperplane-open-webui --create-namespace --values path/to/valuesOverride.yaml
```

### On-Premises Clusters
If Shakudo is deployed to an on-prem cluster with self-signed certificates, Open WebUI requires the certificate to be explicitly provided. The application must trust this certificate otherwise, SSL validation will fail. To address this, the Helm chart includes the `onPrem.enabled` key, which is set to `false` by default. When enabled, it copies the certificate from `onPrem.tlsNamespace` and `onPrem.tlsSecret` into Open WebUI's namespace.

Ensure that the certificate file is named `tls.crt` and exists in the specified `onPrem.tlsNamespace` and `onPrem.tlsSecret`.

Add the following to `valuesOverride.yaml`:
```
onPrem:
  enabled: false
  tlsSecret: ingressgateway-wc-certs
  tlsNamespace: istio-system
```

### Keycloak Protocol
There is an option to choose the communication protocol with Keycloak, either `https` or `http` please add it to `valuesOverride.yaml`:
```
keycloak:
  protocol: <https | http>
```
