# Design Spec: APT Repository 초기 구성 및 배포 설정

본 문서는 GitHub Pages 호스팅 기반의 개인 APT 저장소(`choihunchul/apt-repo`)의 초기 디렉터리 구성 및 문서화 설계를 기술합니다.

---

## 1. 개요 (Overview)
* **목적**: `choihunchul/apt-repo` 저장소를 초기화하고, `publish-apt-repo` 액션이 정상 작동하도록 하는 디렉터리 레이아웃과 상세 가이드를 담은 README.md를 작성합니다.
* **배경**: GitHub Pages를 이용해 개인 CLI 프로젝트의 `.deb` 패키지를 `apt install`로 편리하게 설치하기 위한 미러 저장소 구축을 목표로 합니다.

---

## 2. 범위 (Scope)

### In Scope (구현 범위)
* Git 저장소 초기화 (`git init`)
* 초기 디렉터리 레이아웃 구성:
  * `pool/main/`
  * `dists/stable/main/binary-amd64/`
  * `dists/stable/main/binary-arm64/`
* 빈 디렉터리 추적용 `.gitkeep` 파일 생성
* 임시 placeholder `dists/stable/Release` 파일 생성
* README.md 작성 (GitHub Pages 활성화법, `sources.list` 등록 가이드, `[trusted=yes]` 사유, 연동 가이드 포함)

### Out of Scope (구현 제외 범위)
* 개별 소스 저장소의 `.deb` 파일 빌드 구성
* 소스 저장소의 GitHub Actions 워크플로우 구성 (`publish-apt.yml` 등)
* GitHub PAT (Personal Access Token) 생성 및 등록

---

## 3. 디렉터리 구조 및 파일 상세

### 디렉터리 트리
```text
apt-repo/
├── pool/
│   └── main/
│       └── .gitkeep
└── dists/
    └── stable/
        ├── Release (placeholder)
        └── main/
            ├── binary-amd64/
            │   └── .gitkeep
            └── binary-arm64/
                └── .gitkeep
```

* **`pool/main/`**: 실제 배포되는 `.deb` 파일들이 패키지 첫 글자 단위 디렉터리로 나뉘어 저장되는 공간입니다.
* **`dists/stable/main/binary-amd64/` / `dists/stable/main/binary-arm64/`**: 아키텍처별 `Packages`, `Packages.gz` 파일이 위치할 인덱스 폴더입니다.
* **`dists/stable/Release`**: APT 미러 정보를 명시하는 Release 메타데이터 파일입니다. 초기에는 임시 텍스트가 들어가며, 첫 배포 액션 수행 시 자동으로 재생성 및 업데이트됩니다.

---

## 4. README.md 상세 구성 요구사항

사용자 피드백을 반영하여 아래 핵심 사항들을 포함합니다.

1. **첫 배포 전 주의사항 (Caution)**
   * placeholder `Release` 상태에서 `sources.list` 등록 후 `apt update` 실행 시 오류가 발생할 수 있음을 명시합니다.
   * 첫 배포 성공 후 `curl -I https://choihunchul.github.io/apt-repo/dists/stable/Release` 응답 코드가 200인지 먼저 확인하도록 안내합니다.
2. **`[trusted=yes]` 설명**
   * 해당 저장소는 개인용 미러이며 별도의 GPG 서명이 되어 있지 않으므로, 보안 경고를 우회하기 위해 `[trusted=yes]` 플래그가 필수임을 설명합니다.
3. **패키지 연동 가이드 (Integration)**
   * `choihunchul/github--actions` 레포지토리의 워크플로우 호출 방법 명시:
     * Reusable workflow: `uses: choihunchul/github--actions/.github/workflows/publish-apt-repo.yml@main`
     * 예시 코드 경로: `github--actions/examples/publish-apt-repo-my-cli.yml`
     * 프롬프트 경로: `github--actions/examples/apt-repo-release-prompt.md`
     * Secret 설정: 소스 저장소에 `APT_REPO_TOKEN` (대상 `apt-repo` 저장소에 push 권한이 있는 PAT) 등록 필요.
4. **패키지 목록 표**
   * 배포 현황을 수동으로 기록하는 표를 제공하며, `publish-apt-repo` 실행 성공 후 수동 업데이트함을 명시합니다.

---

## 5. 완료 및 검증 기준 (Definition of Done)
* [ ] 로컬 디렉터리에 상기 트리 구조가 정상적으로 생성됨.
* [ ] README.md 및 `.gitkeep`, placeholder `Release` 파일이 작성됨.
* [ ] `git init` 후 초기 커밋이 정상적으로 수행됨.
* [ ] (원격 푸시 후 수동 확인 항목) GitHub Pages 설정이 완료되고, 첫 배포 액션 작동 후 클라이언트에서 `apt install`이 정상 수행됨.
