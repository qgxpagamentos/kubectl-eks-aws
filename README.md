# Github Action for Kubernetes CLI

Essa Action disponibiliza o comando `kubectl` para Github Actions.

## Uso

`.github/workflows/push.yml`

```yaml
on: push
name: deploy
jobs:
  deploy:
    name: deploy to cluster
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v2

    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-2
    
    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1

    - name: deploy to cluster
      uses: qgxpagamentos/kubectl-aws-eks@master
      env:
        KUBE_CONFIG_DATA: ${{ secrets.KUBE_CONFIG_DATA_STAGING }}
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: my-app
        IMAGE_TAG: ${{ github.sha }}
      with:
        args: set image deployment/$ECR_REPOSITORY $ECR_REPOSITORY=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
        
    - name: verify deployment
      uses: qgxpagamentos/kubectl-aws-eks@master
      env:
        KUBE_CONFIG_DATA: ${{ secrets.KUBE_CONFIG_DATA }}
      with:
        args: rollout status deployment/my-app
```

## Secrets

`KUBE_CONFIG_DATA` – **requerido**: Um arquivo kubeconfig base64-encoded com as credenciais necessárias para o K8s acessar o cluster.
Exemplo de como gerar o arquivo base64-encoded:
```bash
cat $HOME/.kube/config | base64
```
[Inspiration](https://github.com/kodermax/kubectl-aws-eks)