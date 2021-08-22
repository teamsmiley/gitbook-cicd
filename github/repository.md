# Repository

## Source Repository

### Create GitHub or GitLab Account

여기서는 github를 샘플로 사용하겠다.

### create source code repo

프로그램 소스관리

github에서 demo-angular repository를 생성한다.

### create argocd repo

쿠버네티스 배포 관리용

github에서 argocd repository를 생성한다.

### local 에서 demo-angular 프로젝트 생성

angular default project로 사용한다.

```bash
# nodejs가 설치되 있어야 한다.
npm install -g @angular/cli

mkdir ~/Desktop/devops
cd ~/Desktop/devops

ng new demo-angular --routing=true --style=scss

cd demo-angular

git branch -M main
git remote add origin git@github.com:teamsmiley/demo-angular.git
git push -u origin main
```

### demo-argocd 프로젝트 생성

```bash
cd ~/Desktop/devops

mkdir demo-argocd
cd demo-argocd

echo "# demo-argocd" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin git@github.com:teamsmiley/demo-argocd.git
git push -u origin main
```
