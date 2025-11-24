## 1장
### 동일한 머신에서 다른 버전의 Terraform 사용하기
- 도커 컨테이너를 통해 실행하기
- 테라폼 바이너리 설치 파일의 이름을 변경하기
- tfenv를 활용하여 버전 전환하기

여기서는 tfenv를 활용하는 방법을 소개한다.
```bash
#1. Check out tfenv into any path (here is ${HOME}/.tfenv)
git clone --depth=1 https://github.com/tfutils/tfenv.git ~/.tfenv

#2. Add ~/.tfenv/bin to your $PATH any way you like

#bash
echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> ~/.bash_profile

#zsh
echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> ~/.zprofile

#fish
echo 'set -x PATH $HOME/.tfenv/bin $PATH' >> ~/.config/fish/config.fish

#For WSL users
echo 'export PATH=$PATH:$HOME/.tfenv/bin' >> ~/.bashrc
```

tfenv를 통해 특정 버전의 Terraform을 설치하려면 다음과 같은 절차를 따른다.
```bash
# tfenv를 통해 특정 Terraform 버전 설치하기
tfenv install <설치하려는 특정 버전>

# tfenv를 통해 설치할 수 있는 Terraform 버전 확인하기
tfenv list-remote

# tfenv를 통해 설치된 Terraform들의 버전 보기
tfenv list

# 특정 버전의 Terraform 사용하기
tfenv use <특정 버전>

# tfenv를 통해 설치되어 있는 Terraform 특정 버전 삭제하기
tfenv uninstall <특정 버전>

# 실제 사용하려는 Terraform 버전 확인
terrafrom version
```

최신 버전의 Terraform을 통해 상태를 수정한 경우 이전 버전의 Terraform을 통해 상태를 수정할 수 없다.

### Terraform Provider 업그레이드 하기
만약 terraform init 명령어를 수행할 때 문제가 발생한다면, Provider를 업그레이드 해야 할 수 있다.

```bash
# Terraform Provider 업그레이드
terraform init --upgrade
```

.terraform.lock.hcl 파일은 Terraform Provider 버전에 대한 모든 정보를 포함하고 있는 의존성 파일이다. 이 파일을 통해 다른 환경에서도 동일한 버전의 Terraform Provider를 사용하도록 구성할 수 있다.

Terraform Provider를 업그레이드 할 때는 최신 버전의 Terraform Provider가 이전 버전의 Terraform Provider와 달리 지원하지 않는 리소스가 있는지 잘 확인하고, Terraform 스크립트를 수정해야 한다. 그러지 않으면, 예러가 발생한다.

다행히 terraform validate라는 명령어를 통해 미지원 리소스를 확인할 수 있기 때문에, 확인 후 수정하는 것이 바람직하다.


## 2장
Terraform Provider에 별칭을 추가하여, 동일한 Provider를 다른 Provider 처럼 쓸 수 있다.
이러한 구성 방식은, 같은 Terraform Provider에 대해서 여러 구독을 취하고 있을 때 사용하면 효율적이다.

```hcl
provider "azurerm" {
    subscription_id = "xxx"
    alias = "sub1"
    features {}
}

provider "azurerm" {
    subscription_id = "xxx"
    alias = "sub2"
    features {}
}

resource "azurem_resource_group" "example1" {
    provider = azurem.sub1
    name = "rg-sub1"
}

resource "azurem_resource_group" "example2" {
    provider = azurem.sub2
    name = "rg-sub2"
}
```

머신에서 값을 사용하고 스크립트에서는 감추고 싶다. variable 사용
다른 사용자도 해당 값을 공유받고 싶다. locals 사용
Terraform의 실행 결과 값을 출력 받고 싶다. Output 사용

참조를 통해서 암시적 의존성 그래프 확보 가능
직접적인 명시를 통해서 명시적 의존성 그래프 확보 가능(depends_on 사용) 하지만 암시적 의존성을 사용하는 것이 권장됨.

terraform graph 명령어를 사용하면, 해당 Terraform으로 배포되는 리소스 간의 의존성을 그래프로 확인할 수 있다.

