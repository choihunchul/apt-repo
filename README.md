# choihunchul/apt-repo

GitHub Pages를 이용해 여러 개인 CLI 프로젝트의 `.deb` 패키지를 배포하고 설치할 수 있게 제공하는 개인 APT mirror 저장소입니다.

## ⚠️ 첫 배포 전 주의사항
* 저장소에 초기 placeholder `Release` 파일만 존재하고 실제 인덱스(`Packages.gz`)가 생성되기 전에 클라이언트 설정을 등록하면 `apt update` 과정에서 에러가 발생할 수 있습니다.
* 첫 패키지 배포(`publish-apt-repo` 액션 실행) 이후 `curl -I https://choihunchul.github.io/apt-repo/dists/stable/Release` 응답 코드가 **200 OK**로 반환되는지 확인한 후에 아래 클라이언트 설정을 진행하십시오.

## 📦 사용자 설치 (Linux/Debian/Ubuntu)

본 저장소는 GPG 서명을 포함하지 않는 미서명 저장소이므로, APT 보안 경고를 우회하기 위해 `[trusted=yes]` 플래그를 추가하여 등록해야 합니다.

```bash
# 1. APT 소스 리스트 추가 (미서명 저장소 허용 플래그 포함)
echo "deb [trusted=yes] https://choihunchul.github.io/apt-repo stable main" | sudo tee /etc/apt/sources.list.d/choihunchul.list

# 2. 패키지 인덱스 업데이트 및 설치
sudo apt update
sudo apt install <package-name>
```

## ⚙️ GitHub Pages 활성화 방법 (저장소 설정)
1. GitHub의 본 저장소(`choihunchul/apt-repo`)로 이동합니다.
2. **Settings** -> **Pages** 메뉴로 이동합니다.
3. **Build and deployment** -> **Source**에서 `Deploy from a branch`를 선택합니다.
4. **Branch**를 `main` (또는 `master`) 브랜치로 설정하고, 폴더를 `/ (root)`로 지정한 뒤 **Save**를 클릭합니다.

## 🚀 CLI 소스 저장소 연동 가이드

소스 저장소(`{owner}/{project}`)에서 릴리즈 이벤트 발생 시 이 저장소로 패키지를 자동 배포하려면 `choihunchul/github--actions` 공통 액션을 연동해야 합니다.

### 1. Secret 설정
* 소스 저장소의 **Settings** -> **Secrets and variables** -> **Actions**에 `APT_REPO_TOKEN` 시크릿을 등록합니다.
* `APT_REPO_TOKEN`은 `choihunchul/apt-repo` 저장소에 대한 **Contents: Read and write** 권한(또는 `repo` classic scope)을 지닌 PAT(Personal Access Token)이어야 합니다.

### 2. GitHub Actions 워크플로우 구성 예시
소스 저장소의 `.github/workflows/publish-apt.yml` 파일에서 reusable workflow를 호출합니다.

```yaml
name: Publish APT Repository

on:
  push:
    tags:
      - "v*"
  workflow_run:
    workflows: ["Release"]
    types: [completed]
  workflow_dispatch:
    inputs:
      version:
        description: "Release version, e.g. 1.0.0 or v1.0.0"
        required: true

permissions:
  contents: read

jobs:
  publish-apt:
    if: ${{ github.event_name != 'workflow_run' || github.event.workflow_run.conclusion == 'success' }}
    uses: choihunchul/github--actions/.github/workflows/publish-apt-repo.yml@main
    with:
      apt-repo: choihunchul/apt-repo
      package-name: <my-cli>  # 배포 패키지 이름으로 변경
      asset-amd64: <my-cli>_${{ github.event_name == 'workflow_dispatch' && inputs.version || github.ref_name }}_amd64.deb
      asset-arm64: <my-cli>_${{ github.event_name == 'workflow_dispatch' && inputs.version || github.ref_name }}_arm64.deb
      version: ${{ github.event_name == 'workflow_dispatch' && inputs.version || '' }}
      verify-tag: ${{ github.event_name != 'workflow_dispatch' }}
    secrets:
      APT_REPO_TOKEN: ${{ secrets.APT_REPO_TOKEN }}
```

* **참고 문서**:
  * Reusable 워크플로우 예시: `github--actions/examples/publish-apt-repo-my-cli.yml`
  * AI 자동 배포 가이드 프롬프트: `github--actions/examples/apt-repo-release-prompt.md`

## 📋 배포 패키지 목록
*(배포 성공 시 수동으로 갱신하는 테이블입니다)*

| 패키지명 | 아키텍처 | 최신 버전 | 설명 |
| :--- | :--- | :--- | :--- |
| - | - | - | - |
