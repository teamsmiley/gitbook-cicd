# notification slack

{% embed url="https://argocd-notifications.readthedocs.io/en/stable/" caption="" %}

## slack 설정

[https://argocd-notifications.readthedocs.io/en/stable/services/slack/](https://argocd-notifications.readthedocs.io/en/stable/services/slack/)

slack에서 add apps  ==&gt; app directory ==&gt; build ==&gt; create custom app 

![](../.gitbook/assets/image%20%285%29.png)



![](../.gitbook/assets/image%20%289%29.png)

![](../.gitbook/assets/image%20%284%29.png)

![](../.gitbook/assets/image%20%2810%29.png)

create an app from scratch

![](../.gitbook/assets/image%20%287%29.png)

![](../.gitbook/assets/image%20%286%29.png)

create app



oauth permission

![](../.gitbook/assets/2021-05-24-17-38-58.png)

bot token scope ==&gt; add an oauth scope ==&gt; chat:write추

![](../.gitbook/assets/2021-05-24-17-41-55.png)

![](../.gitbook/assets/2021-05-24-17-43-39.png)

chat:write:customize는 메세지를 커스터마이즈하려면 추

 Install App to Workspace

![](../.gitbook/assets/image%20%2811%29.png)

allow

토큰이 생성이 되면 복사해둔다.

![](../.gitbook/assets/2021-05-24-17-46-26.png)

채널을 만든다. argocd 

채널에서 Add apps &gt;&gt; argocd-notifications

![](../.gitbook/assets/image%20%282%29.png)

![](../.gitbook/assets/image%20%283%29.png)







core/argocd/

{% embed url="https://ingress.yml" caption="" %}

slack-notification/

{% embed url="https://kustomization.yaml" caption="" %}

{% embed url="https://slack-configmap.yml" caption="" %}

{% embed url="https://slack-secret.yml" caption="" %}

