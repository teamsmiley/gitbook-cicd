# export and variable

## 쉘 변수

```bash
myvar="aaa"
echo $myvar
unset myvar #삭제
echo $myvar

echo $SHELL
```

## export

export 명령어를 통해서 쉘 변수를 환경변수로 저장

```bash
env # aws_profile이 없다.

export AWS_PROFILE=myprofile

env # aws_profile이 생겼다.
```

환경변수가 새로운 터미널을 열면 없어진다.

항상 열고 싶으면 ~/.bashrc 나 ~/.zshrc에 추가해주면 된다.

{% tabs %}
{% tab title="~/.zshrc" %}
```bash
vi ~/.zshrc

...
export AWS_PROFILE=myprofile # 추가
```
{% endtab %}
{% endtabs %}

새창을 열고 값을 echo로 확인해본다.

이게 되는 이유는 새창을 열때마다. ~/.zshrc가 실행되기 때문이다.

