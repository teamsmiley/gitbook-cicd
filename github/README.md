# GitHub

GitHub Action을 이용하여 이미지를 빌드해보자.

다음 글을 읽고 내용은 한번 보고 위 글에 이어서 진행해 보자.

```bash
cd ~/Desktop/demo-angular
mkdir -p .github/workflows
touch .github/workflows/build.yml
```

{% tabs %}
{% tab title=".github/workflows/build.yml" %}

```yaml
name: CI-CD

on:
  push:
    branches: [main, dev]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: AKIA4LL7IJKQBE34KJW7
          aws-secret-access-key: 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
          aws-region: us-west-2

      - name: Login to Amazon ECR
        id: ecr-user
        uses: aws-actions/amazon-ecr-login@v1

      - name: Build, tag, and push staging image to Amazon ECR
        if: github.ref == 'refs/heads/dev'
        env:
          ECR_REGISTRY: 849059793568.dkr.ecr.us-west-2.amazonaws.com
          ECR_REPOSITORY: www
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -f Dockerfile-dev .
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG

      - name: Build, tag, and push production image to Amazon ECR
        if: github.ref == 'refs/heads/main'
        env:
          ECR_REGISTRY: 849059793568.dkr.ecr.us-west-2.amazonaws.com
          ECR_REPOSITORY: www
          IMAGE_TAG: ${{ github.sha }}
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG -f Dockerfile-prod .
          docker tag $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:latest
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest

      - name: Checkout argocd
        uses: actions/checkout@v2
        with:
          repository: stanleyprep/argocd
          token: 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

      - name: replace staging image version number to sha
        if: github.ref == 'refs/heads/dev'
        run: |
          cd apps/www-staging
          sed "s/:latest/:${{ github.sha }}/g" 21.deployment.origin > 21.deployment.yml

      - name: replace production image version number to sha
        if: github.ref == 'refs/heads/main'
        run: |
          cd apps/www-prod
          sed "s/:latest/:${{ github.sha }}/g" 21.deployment.origin > 21.deployment.yml

      - name: Commit files
        run: |
          git config --local user.email "action@github.com"
          git config --local user.name "GitHub Action"
          git commit -am "change docker tag"
          git push
```

{% endtab %}

{% endtabs %}
