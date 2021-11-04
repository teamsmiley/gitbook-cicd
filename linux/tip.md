# tip

## zsh

```sh
sudo bash
yum install zsh -y

chsh -s /bin/zsh root
```

## ohmyzsh

```sh
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"

cd ~/.oh-my-zsh/custom/plugins/
git clone https://github.com/zsh-users/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git

vi ~/.zshrc

plugins=(git zsh-syntax-highlighting zsh-autosuggestions kubectl kube-ps1) #추가한다.
```

<https://github.com/ohmyzsh/ohmyzsh/tree/master/plugins/kubectl>

## kubectl

```sh
cd
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
mv ./kubectl /usr/local/bin/kubectl
kubectl version --client

source <(kubectl completion zsh)
```

## k9s

https://snapcraft.io/install/k9s/centos

```sh
sudo yum install epel-release
sudo yum install snapd
sudo systemctl enable --now snapd.socket
sudo ln -s /var/lib/snapd/snap /snap
sudo snap install k9s
```

## .zshrc

```conf

plugins=(git kubectl kube-ps1 zsh-syntax-highlighting zsh-autosuggestions)

NEWLINE=$'\n'
export PROMPT='[$FG[154]%T%{$reset_color%}][%n][%{$fg[cyan]%}%m %{$reset_color%}%~] $(git_prompt_info)${NEWLINE}# '

export PATH=~/.local/bin/:$PATH

source <(kubectl completion zsh)

bindkey -v
```
