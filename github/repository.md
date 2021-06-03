# Repository

GitHub Action을 이용하여 이미지를 빌드해보자.

다음 글을 읽고 내용은 한번 보고 위 글에 이어서 진행해 보자.

```bash
cd ~/Desktop/demo-angular
mkdir -p .github/workflows
touch .github/workflows/build.yml
```

{% code title=".github/workflows/build.yml" %}

```yaml
- name: Checkout
  uses: actions/checkout@v2

- name: Node 12.x
  uses: actions/setup-node@v2.1.5
  with:
    node-version: '12.x'

- name: install @angular/cli && npm install && ng build
  run: |
    sudo npm install -g @angular/cli
    npm install
    ng build --prod
```

{% endcode %}
