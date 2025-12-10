# MkDocs 사용 가이드

MkDocs는 Markdown 형식으로 작성한 문서를 아름답고 일관된 스타일의 정적 웹사이트로 변환해주는 도구입니다. 이 가이드는 수업용 문서 사이트를 처음 세팅하는 사람을 기준으로, 설치부터 GitHub Pages 배포까지의 전체 흐름을 단계별로 설명합니다.[^1][^2][^3]

## 설치

MkDocs는 Python 기반 도구이므로 시스템에 Python 인터프리터가 설치되어 있어야 합니다. 이 프로젝트는 최신 언어 기능과 보안 업데이트를 활용하기 위해 Python 3.13 이상을 요구합니다. Python 버전 관리를 위해 `pyenv`, `venv`, 혹은 `uv`의 Python 관리 기능을 함께 사용하는 것도 좋습니다. 여러 수업용 프로젝트를 병렬로 운영한다면, 각 프로젝트마다 독립된 Python 버전과 가상 환경을 유지하면 충돌을 줄일 수 있습니다.

이 프로젝트는 `uv`를 프로젝트 관리자로 사용합니다. `uv`는 Rust로 구현된 매우 빠른 Python 패키지 및 프로젝트 관리자이며, 의존성 관리와 가상 환경 생성, Python 버전 설치까지 한 도구로 통합 관리할 수 있습니다.[^8][^9]

일반적인 `pip + venv` 조합에 비해, `uv`는 다음과 같은 장점이 있습니다.

- 프로젝트별 잠금 파일(락파일)을 통해 재현 가능한 의존성 환경을 보장
- 의존성 해석 및 설치 속도가 매우 빠름
- Python 버전 설치와 전환을 하나의 CLI에서 처리 가능
- `pyproject.toml` 기반의 현대적인 프로젝트 구성을 지원

#### 1. uv 설치 확인

먼저 `uv`가 설치되어 있는지 확인합니다.

```bash
uv --version
```

버전 정보가 출력되면 정상 설치된 것입니다. 만약 명령어를 찾을 수 없다는 메시지가 뜬다면 아직 설치되지 않은 상태입니다.

`uv`가 설치되어 있지 않다면, 다음 명령어로 설치할 수 있습니다.

```bash
scoop install uv # scoop
winget install -e --id astral-sh.uv # winget
curl -LsSf https://astral.sh/uv/install.sh | sh #macOS/Linux
```

설치가 끝난 뒤에는 새 터미널을 열고 다시 `uv --version`을 실행해서 PATH 설정이 제대로 적용되었는지 확인합니다.

#### 2. 프로젝트 의존성 설치

프로젝트 루트 디렉터리에서 다음 명령어를 실행하여 필요한 패키지를 설치합니다.

```bash
uv sync
```

이 명령어는 `pyproject.toml`과 `uv.lock` 파일을 읽어, 지정된 버전의 패키지들을 설치하고 가상 환경까지 자동으로 구성합니다. 이때, 기존에 다른 가상 환경을 수동으로 활성화할 필요는 없으며, `uv`가 내부적으로 관리하는 환경을 사용하게 됩니다.

이 명령어는 다음과 같은 패키지들을 설치합니다.

```bash
uv mkdocs mkdocs-rtd-dropdown pymdown-extensions
```

- `mkdocs` (>=1.6.1): MkDocs 핵심 패키지.
- `mkdocs-rtd-dropdown` (>=1.0.2): ReadTheDocs 테마에 드롭다운 메뉴 기능을 추가하는 테마 확장.
- `pymdown-extensions` (>=10.0): Admonition, Details, 코드 블록 강화 등 다양한 Markdown 확장을 제공.

의존성을 수정했을 때(예: 플러그인 추가)도 `uv sync`를 다시 실행하면 필요한 패키지가 자동으로 갱신됩니다.

#### 3. 설치 확인

설치가 완료되면 다음 명령어로 MkDocs가 제대로 설치되었는지 확인합니다.

```bash
uv run mkdocs --version
```

`uv run`은 해당 프로젝트에 연결된 가상 환경을 자동으로 활성화한 뒤 명령을 실행하기 때문에, 별도의 `source venv/bin/activate` 단계가 필요하지 않습니다.

## 글쓰기

### 기본 구조 이해

MkDocs 프로젝트의 기본 구조는 다음과 같습니다.

```text
practice-mkdocs/
├── docs/              # 문서 소스 파일 (Markdown)
│   └── index.md
├── mkdocs.yml         # MkDocs 설정 파일
├── pyproject.toml     # Python 프로젝트 설정
└── site/              # 빌드된 정적 사이트 (자동 생성)
```

MkDocs는 기본적으로 `docs/` 디렉터리를 문서 루트로 사용하며, `mkdocs.yml`에서 `docs_dir` 옵션을 변경해 다른 디렉터리를 사용할 수도 있습니다. `site/` 디렉터리는 빌드할 때마다 새로 생성되며, 보통 Git 버전 관리에서 제외(`.gitignore`)하는 것이 좋습니다.

### Markdown 파일 작성

#### 1. 새 문서 생성

`docs/` 디렉터리 내에 `.md` 확장자를 가진 파일을 생성합니다.

```bash
# docs/getting-started/new-topic.md 파일 생성
```

실제 작업에서는 편리한 Markdown 에디터(VS Code, Typora 등)를 사용해 파일 생성과 편집을 동시에 진행하는 것이 좋습니다. 파일을 만든 뒤에는 `mkdocs.yml`의 `nav` 항목에 추가해야 메뉴에 표시됩니다.

#### 2. 기본 Markdown 문법

MkDocs는 표준 Markdown 문법을 지원하며, `pymdown-extensions`를 통해 확장된 문법도 사용할 수 있습니다.

```markdown
# 제목 1
## 제목 2
### 제목 3

굵은 글씨와 *기울임*을 사용할 수 있습니다.

- 리스트 항목 1
- 리스트 항목 2
  - 중첩 리스트

1. 번호 있는 리스트
2. 두 번째 항목

[링크 텍스트](https://example.com)

![이미지 설명](https://images.unsplash.com/photo-1723707303970-9e8e73721dd0)

`인라인 코드`를 사용할 수 있습니다.

```
fn main() {
    println!("Hello, World!");
}
```

> 인용문을 작성할 수 있습니다.

```

문서 스타일을 일정하게 유지하기 위해, 헤딩 깊이(예: 최대 H3까지)와 코드 블록 언어 표기 규칙을 팀 내에서 미리 정해 두는 것을 추천합니다.

#### 3. 확장 기능 활용

이 프로젝트는 `pymdown-extensions`를 사용하여 다양한 확장 기능을 제공합니다.

##### Admonition (주의/경고 박스)

중요한 참고사항, 주의, 경고, 팁 등을 시각적으로 구분하고 싶을 때 사용합니다.

```markdown
!!! note "참고"
    이것은 참고 사항입니다.

!!! warning "주의"
    이것은 주의가 필요한 내용입니다.

!!! danger "위험"
    이것은 위험한 작업입니다.

!!! tip "팁"
    이것은 유용한 팁입니다.
```

실습에서 자주 틀리는 부분을 `warning`, 시스템에 영향을 줄 수 있는 명령을 `danger`로 표시하면 학습자가 리스크를 빠르게 인지할 수 있습니다.

##### Details (접을 수 있는 섹션)

부가 설명, 심화 내용, 스포일러 등을 감추고 싶을 때 사용합니다.

```markdown
??? note "클릭하여 펼치기"
    숨겨진 내용이 여기에 표시됩니다.
    
    여러 줄의 내용도 포함할 수 있습니다.
```

이 기능은 "배경 이론", "추가 참고자료"처럼 처음 읽을 때는 건너뛰어도 되는 내용을 포함할 때 유용합니다.

##### 코드 블록 강화

코드 블록에 언어를 지정하면 자동으로 문법 강조가 적용됩니다.

```markdown
// Rust 코드는 자동으로 문법 강조됩니다
fn main() {
    println!("Hello, World!");
}
```

```markdwon
# Bash 스크립트도 지원됩니다
echo "Hello, World!"
```

프로젝트 설정에 따라 `rust`, `toml`, `bash`, `makefile` 등의 언어 하이라이팅이 활성화되어 있습니다. 코드 예제를 제공할 때는 실제로 컴파일/실행 가능한 최소 예제를 넣어두면, 학습자가 그대로 복사해 실습하기 좋습니다.

##### 테이블

명령어 요약, 옵션 비교, 환경 차이 등을 설명할 때 표를 사용하면 가독성이 높아집니다.

```markdown
| 열1   | 열2   | 열3   |
|-------|-------|-------|
| 데이터1 | 데이터2 | 데이터3 |
| 데이터4 | 데이터5 | 데이터6 |
```

## 네비게이션 설정

문서의 구조와 메뉴는 `mkdocs.yml` 파일의 `nav` 섹션에서 정의합니다. `nav`를 생략하면 파일 시스템 순서대로 자동 네비게이션이 생성되지만, 수업용 사이트에서는 의미 있는 순서와 그룹을 만들기 위해 `nav`를 명시하는 것이 일반적입니다.

### 현재 설정 예시

```yaml
nav:
  - 홈: index.md
  - 시작하기:
    - 개요: getting-started/overview.md
```

여기서 상위 항목(`시작하기`)는 섹션 이름이고, 그 아래 들여쓴 항목들이 실제 문서 파일에 매핑됩니다.

### 새 문서 추가하기

1. 파일 생성: `docs/` 디렉터리에 새 Markdown 파일을 생성합니다.

2. 네비게이션에 추가: `mkdocs.yml` 파일의 `nav` 섹션에 새 문서를 추가합니다.

```yaml
nav:
  - 홈: index.md
  - 시작하기:
    - 개요: getting-started/overview.md
```

3. 저장 후 확인: 개발 서버(`mkdocs serve`)가 실행 중이면 파일 저장 후 브라우저에서 메뉴가 자동으로 갱신됩니다.

`nav`에 포함되지 않은 Markdown 파일은 URL로 직접 접근할 수는 있지만, 사이드바 메뉴에는 나타나지 않으므로, 학습자가 접근해야 하는 문서는 모두 `nav`에 등록하는 것이 좋습니다.

### 네비게이션 구조 팁

1. 계층 구조: 들여쓰기를 사용해 상·하위 메뉴를 구성하고, 너무 깊은 중첩(4단계 이상)은 피하는 것이 좋습니다.
2. 섹션 제목: 메뉴 항목에 파일 경로 없이 제목만 지정하면 섹션 헤더로만 사용되며, 실제 페이지는 연결되지 않습니다.
3. 순서 제어: `nav`에 나열된 순서 그대로 메뉴가 표시되므로, "학습 순서"에 맞게 파일을 배치할 수 있습니다.
4. 일관된 명명: 폴더 구조와 `nav` 이름을 어느 정도 맞춰 두면, 유지보수 시 파일 위치를 추적하기 쉬워집니다.

## 이미지 추가

이미지는 `docs/` 디렉터리 내에 `img/` 폴더를 만들어 정리하는 것을 권장합니다.

```text
docs/
├── img/
│   └── screenshot.png
└── guide/
    └── tutorial.md
```

문서에서 이미지를 참조할 때는 아래와 같습니다.

```markdown
![설명 텍스트](img/screenshot.png)
```

이미지 파일명은 영문 소문자와 하이픈을 사용하는 것이 URL 호환성 측면에서 안전합니다. 스크린샷이 많아질 경우 `img/setup/`, `img/boot/`와 같이 하위 폴더로 기능별로 분류하면 관리가 수월합니다.

## 내부 링크

문서 간 링크를 만들 때는 파일 경로를 사용합니다.

```markdown
[초기 설정 가이드](getting-started/setup.md)
[프로젝트 구조 섹션](guide/project-structure.md)
```

특정 제목으로 링크하려면

```markdown
[설정 섹션](getting-started/setup.md#설정-섹션)
```

MkDocs는 기본적으로 Markdown 헤딩을 소문자로 변환하고, 공백을 하이픈(`-`)으로 치환해 앵커 ID를 생성합니다. 한글 헤딩도 브라우저가 URL 인코딩을 통해 처리하므로 동작하지만, 필요하다면 HTML ID를 직접 지정해 더 명확하게 관리할 수 있습니다.

## 확인

### 개발 서버 실행

문서를 작성하는 동안 실시간으로 결과를 확인하려면 개발 서버를 실행합니다.

```bash
uv run mkdocs serve
```

이 명령어는 로컬 웹 서버를 띄우고, 문서 파일 변경을 감지해 자동으로 다시 빌드합니다. 대규모 문서가 아니면 빌드 시간은 보통 1초 내외로 매우 짧습니다.

### 브라우저에서 확인

브라우저를 열고 다음 주소로 접속합니다.

```text
http://localhost:8000/
```

여러 명이 같은 네트워크에서 문서를 함께 보고 싶다면, "외부 접근 허용" 방식으로 서버를 띄우고, 로컬 IP 주소를 공유하면 됩니다.

### 개발 서버의 장점

- 자동 새로고침: Markdown 파일을 저장하면 브라우저가 자동으로 새로고침됩니다.
- 실시간 미리보기: 테마, 네비게이션, 코드 하이라이팅 등을 실제 페이지와 거의 동일한 형태로 확인 가능합니다.
- 빠른 피드백: Markdown 문법 오류나 설정 오류가 있으면 터미널 로그로 바로 확인할 수 있습니다.

### 서버 중지

서버를 중지하려면 터미널에서 `Ctrl + C`를 누릅니다. 하나의 포트에서 이미 서버가 실행 중인 상태에서 다시 `mkdocs serve`를 실행하면 포트 충돌 에러가 발생할 수 있습니다.

### 특정 포트 사용

기본 포트(8000)가 사용 중이면 다른 포트를 지정할 수 있습니다.

```bash
uv run mkdocs serve -a 127.0.0.1:8080
```

팀 내에서 여러 MkDocs 프로젝트를 동시에 띄워두고 비교할 때, 포트를 각각 다르게 지정하면 편리합니다.

### 외부 접근 허용

같은 네트워크의 다른 기기에서 접근하려면

```bash
uv run mkdocs serve -a 0.0.0.0:8000
```

이렇게 하면 로컬 네트워크 상의 다른 PC, 태블릿, 스마트폰에서도 IP 주소를 통해 문서를 열 수 있습니다. 단, 방화벽 설정에 따라 접근이 차단될 수 있으므로, 개발 환경에서만 사용하는 것을 권장합니다.

*

## 배포

### 사이트 빌드

배포하기 전에 정적 사이트를 빌드합니다.

```bash
uv run mkdocs build
```

이 명령어는 `mkdocs.yml` 설정을 읽고, `docs/` 디렉터리의 Markdown 파일들을 HTML, CSS, JavaScript가 포함된 정적 사이트로 변환합니다.

### 빌드 결과 확인

빌드가 완료되면 `site/` 디렉터리의 내용을 확인할 수 있습니다.

```bash
# Windows PowerShell
dir site

# Linux/macOS
ls site
```

`index.html`, `search/`, `assets/` 등의 파일과 폴더가 생성되며, 이는 그대로 웹 서버의 루트 디렉터리에 업로드할 수 있는 형태입니다.

### 로컬에서 빌드 결과 테스트

빌드된 사이트를 로컬에서 테스트하려면:

```bash
cd site
python -m http.server 8000
```

이렇게 하면 정적 파일만을 사용하는 실제 배포 환경과 거의 동일한 조건에서 테스트할 수 있습니다. 별도의 HTTP 서버(Nginx, Caddy 등)를 사용 중이라면 해당 서버로 `site/` 디렉터리를 루트로 설정해 테스트할 수도 있습니다.

### 수동 배포

빌드된 `site/` 디렉터리의 내용을 웹 서버에 업로드하면 됩니다.

1. `site/` 디렉터리의 모든 파일을 복사합니다.
2. 웹 호스팅 서비스의 루트 디렉터리에 업로드합니다.
3. 웹사이트 URL로 접속하여 확인합니다.

S3, Netlify, Vercel 등 정적 사이트 호스팅 서비스도 MkDocs로 생성한 결과물을 그대로 지원합니다.

### 빌드 옵션

#### 깨끗한 빌드

```bash
uv run mkdocs build --clean
```

이 옵션은 파일 이름이 변경되었을 때 이전 파일이 남는 문제를 방지하는 데 유용합니다. 링크 깨짐, 누락된 파일 등 잠재적인 문제를 CI 단계에서 바로 잡고 싶다면 `--strict` 옵션을 기본으로 사용하는 것이 좋습니다.

#### 사이트 디렉터리 변경

기본 `site/` 대신 다른 디렉터리에 빌드하는 방법은 아래와 같습니다.

```bash
uv run mkdocs build -d output/
```

CI 환경에서 빌드 아티팩트 디렉터리를 분리하거나, 여러 버전의 문서를 동시에 관리할 때 유용합니다.

## GitHub 페이지(`ci.yml`)

이 프로젝트는 GitHub Actions를 사용하여 자동으로 GitHub Pages에 배포합니다. 수동으로 `mkdocs gh-deploy`를 실행할 필요 없이, `main` 브랜치에 변경사항을 push하면 CI가 자동으로 새 버전을 배포합니다.

### CI/CD 워크플로우 이해

`.github/workflows/ci.yml` 파일이 자동 배포를 담당합니다.

```yaml
name: CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

permissions:
  contents: write
  pages: write
  id-token: write

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: astral-sh/setup-uv@v7
      - uses: actions/setup-python@v6
      - run: uv sync
      - run: uv run mkdocs gh-deploy --force
```

`astral-sh/setup-uv` 액션은 CI 환경에서 `uv` 바이너리를 설치하고, `uv sync`와 `uv run`을 사용할 수 있게 설정합니다. `mkdocs gh-deploy`는 내부적으로 `ghp-import`를 사용해 `gh-pages` 브랜치에 빌드 결과를 커밋하고 푸시합니다.

### 워크플로우 동작 방식

1. 트리거: `main` 브랜치에 push하거나, `main`을 대상으로 하는 pull request가 생성되면 실행됩니다.
2. 환경 설정: Ubuntu 최신 버전에서 Python과 `uv`를 설치합니다.
3. 의존성 설치: `uv sync`로 프로젝트 의존성을 설치하고 가상 환경을 구성합니다.
4. 배포: `uv run mkdocs gh-deploy --force`로 문서를 빌드하고 `gh-pages` 브랜치에 배포합니다.

필요하다면 `gh-deploy` 실행 전에 `uv run mkdocs build --strict`를 한 번 더 실행해, 빌드가 실패하면 배포를 막도록 구성할 수 있습니다.

### GitHub Pages 설정

#### 1. 저장소 설정 확인

GitHub 저장소의 Settings > Pages에서 다음을 확인합니다.

- Source: "GitHub Actions" 선택
- Branch: `gh-pages` 브랜치 (자동 생성됨)

MkDocs의 `gh-deploy`는 기본적으로 `gh-pages` 브랜치를 생성하고, 그 브랜치를 GitHub Pages의 소스로 사용합니다.

#### 2. 권한 설정

워크플로우 파일에 다음 권한이 설정되어 있어야 합니다.

```yaml
permissions:
  contents: write    # 저장소에 쓰기 권한
  pages: write       # GitHub Pages에 쓰기 권한
  id-token: write    # OIDC 토큰 사용 권한
```

또한, 저장소 Settings > Actions > General에서 워크플로우 권한이 "Read and write permissions"로 설정되어 있어야 `gh-pages` 브랜치에 푸시할 수 있습니다.

### 배포 확인

배포가 완료되면 다음 URL로 접속할 수 있습니다.

```text
https://[사용자명].github.io/[저장소명]/
```

예를 들어

```text
https://sigma.github.io/os-make/
```

GitHub Actions의 배포 작업이 끝난 뒤 실제 페이지가 보이기까지 수 분 정도 지연이 있을 수 있으니, 잠시 기다린 뒤 브라우저 캐시를 비우고 다시 접속해 보아야 합니다.

### 문제 해결

#### 배포가 실패하는 경우

1. 워크플로우 로그 확인: GitHub 저장소의 Actions 탭에서 실패한 워크플로우를 선택해 상세 로그를 확인합니다.
2. 권한 확인: 저장소 Settings > Actions > General에서 워크플로우 권한이 쓰기 권한을 포함하는지 확인합니다.
3. 의존성 확인: `pyproject.toml`의 의존성이 올바른지, `uv sync`가 CI에서도 정상 작동하는지 확인합니다.
4. 로컬 테스트: 로컬에서 `uv run mkdocs build --strict`를 실행해 빌드 에러가 없는지 먼저 확인합니다.

#### 사이트가 업데이트되지 않는 경우

- GitHub Pages 배포에는 몇 분이 걸릴 수 있습니다.
- 브라우저 캐시를 지우고 강력 새로고침(`Ctrl + Shift + R` 또는 `Cmd + Shift + R`)을 시도합니다.
- `gh-pages` 브랜치의 최신 커밋이 CI에서 생성한 시점과 일치하는지 확인합니다.

*

### 커스터마이징

#### 다른 브랜치에 배포

`ci.yml` 파일을 수정하여 여러 브랜치에서 문서를 배포하도록 할 수 있습니다.

```yaml
on:
  push:
    branches: [ "main", "develop" ]  # 여러 브랜치 추가
```

이 경우, 브랜치마다 다른 URL 하위 경로를 사용하거나, 별도의 저장소를 두는 등 구조를 명확히 해야 합니다.

#### 배포 전 테스트

배포 전에 빌드가 성공하는지 확인하는 단계를 추가할 수 있습니다.

```yaml
steps:
  - uses: actions/checkout@v5
  - uses: astral-sh/setup-uv@v7
  - uses: actions/setup-python@v6
  - run: uv sync
  - run: uv run mkdocs build --strict  # 배포 전 빌드 테스트
  - run: uv run mkdocs gh-deploy --force
```

이렇게 하면 빌드에 실패한 커밋이 GitHub Pages에 반영되는 것을 방지할 수 있습니다.

## 추가 팁

### 검색 기능

MkDocs는 기본적으로 클라이언트 사이드 검색 기능을 제공합니다. `search` 플러그인이 기본 활성화되어 있으며, 별도 설정이 없어도 상단 또는 사이드바에 검색 창이 표시됩니다.

### 테마 커스터마이징

현재 프로젝트는 `readthedocs` 테마를 사용합니다. 다른 테마로 변경하려면 `mkdocs.yml`을 수정합니다.

```yaml
theme:
  name: readthedocs
```

Material 테마 등 다른 테마를 사용하려면 패키지를 설치한 뒤 이름을 변경합니다.

```yaml
theme:
  name: material  # Material 테마로 변경
```

테마마다 지원하는 옵션이 다르므로, 사용 중인 테마 문서를 참고해 색상, 폰트, 로고, 푸터 등을 커스터마이징할 수 있습니다.

### 플러그인 사용

추가 기능을 위해 플러그인을 설치하고 활성화할 수 있습니다.

1. `pyproject.toml`에 플러그인 패키지 추가
2. `uv sync` 실행
3. `mkdocs.yml`에 플러그인 설정 추가

예시:

```yaml
plugins:
  - search
  - minify:
      minify_html: true
```

`minify` 플러그인은 HTML을 압축해 페이지 로딩 속도를 향상시킬 수 있으며, `git-revision-date`와 같은 플러그인은 각 문서의 마지막 수정일을 표시할 수 있습니다.

### 다국어 지원

여러 언어로 문서를 제공하려면 `mkdocs-material` 테마와 `i18n` 관련 플러그인을 함께 사용할 수 있습니다. 언어별로 디렉터리를 분리하거나, 언어 스위처를 통해 하나의 사이트에서 다국어를 전환하는 방식도 지원됩니다.

### 버전 관리

문서의 여러 버전을 관리하려면 MkDocs의 플러그인 생태계를 활용할 수 있습니다. 예를 들어, Git 브랜치나 태그를 기반으로 버전별 문서를 제공하는 플러그인이나, 각 페이지의 변경 이력을 표시해 주는 플러그인이 있습니다. 커널 개발 수업처럼 "학기별 버전"을 남기고 싶다면, 브랜치 전략과 함께 문서 버전 플러그인을 고려해 볼 수 있습니다.

## 참고 자료

- [MkDocs 공식 문서](https://www.mkdocs.org/)
- [MkDocs 사용자 가이드](https://www.mkdocs.org/user-guide/)
- [MkDocs 배포 가이드](https://www.mkdocs.org/user-guide/deploying-your-docs/)
- [Python-Markdown 확장](https://python-markdown.github.io/extensions/)
- [PyMdown Extensions 문서](https://facelessuser.github.io/pymdown-extensions/)
- [uv 공식 문서](https://docs.astral.sh/uv/)
- [GitHub Pages 문서](https://docs.github.com/en/pages)
- [GitHub Actions 문서](https://docs.github.com/en/actions)

[^1]: https://www.mkdocs.org/user-guide/installation/
[^2]: https://www.mkdocs.org/user-guide/writing-your-docs/
[^3]: https://www.mkdocs.org/user-guide/deploying-your-docs/
[^8]: https://docs.astral.sh/uv/
[^9]: https://github.com/astral-sh/uv/blob/main/mkdocs.template.yml
