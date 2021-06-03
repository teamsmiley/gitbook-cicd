# notification slack

{% embed url="https://argocd-notifications.readthedocs.io/en/stable/" caption="" %}

## slack 설정

[https://argocd-notifications.readthedocs.io/en/stable/services/slack/](https://argocd-notifications.readthedocs.io/en/stable/services/slack/)

slack에서 add apps ==&gt; app directory ==&gt; build ==&gt; create custom app

![](../.gitbook/assets/argocd-notifications-01.png)

![](../.gitbook/assets/argocd-notifications-02.png)

![](../.gitbook/assets/argocd-notifications-03.png)

![](../.gitbook/assets/argocd-notifications-04.png)

create an app from scratch

![](../.gitbook/assets/argocd-notifications-05.png)

![](../.gitbook/assets/argocd-notifications-06.png)

create app

oauth permission

![](../.gitbook/assets/argocd-notifications-07.png)

bot token scope ==&gt; add an oauth scope ==&gt; chat:write추

![](../.gitbook/assets/argocd-notifications-08.png)

![](../.gitbook/assets/argocd-notifications-09.png)

chat:write:customize는 메세지를 커스터마이즈하려면 추가

Install App to Workspace

![](../.gitbook/assets/argocd-notifications-10.png)

allow

토큰이 생성이 되면 복사해둔다.

![](../.gitbook/assets/argocd-notifications-11.png)

argocd 채널을 만든다.

채널에서 Add apps &gt;&gt; argocd-notifications

![](../.gitbook/assets/argocd-notifications-12.png)

![](../.gitbook/assets/argocd-notifications-13.png)
