# notification

{% embed url="https://argocd-notifications.readthedocs.io/en/stable/" caption="" %}

## slack 설정

slack에서 app 추가 ==> custom app build를 눌러서 해보자.

https://argocd-notifications.readthedocs.io/en/stable/services/slack/

![](./images/2021-05-24-17-37-37.png)

oauth permission

![](./images/2021-05-24-17-38-58.png)

bot token scope

![](./images/2021-05-24-17-41-55.png)

![](./images/2021-05-24-17-43-39.png)

chat:write:bot

Scroll back to the top, click 'Install App to Workspace' button and confirm the installation.

![](./images/2021-05-24-17-45-39.png)

![](./images/2021-05-24-17-46-05.png)

allow

![](./images/2021-05-24-17-46-26.png)

xoxb-1479752548402-2099093888259-sl1yj1ypSN2ouGFYraOJZmYH

Create a public or private channel, for this example my_channel

Add your bot to this channel otherwise it won't work

채널에서 Add app

![](./images/2021-05-24-17-51-00.png)

argo-notify

Store token in argocd_notifications-secret Secret

token을 복사해둔다.
create folder

core/argocd/

{% embed url="https://ingress.yml" %}

slack-notification/

{% embed url="https://kustomization.yaml" %}

{% embed url="https://slack-configmap.yml" %}

{% embed url="https://slack-secret.yml" %}
