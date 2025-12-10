# MKDocs 연습

> MKDocs를 활용한 정적 페이지 배포 관련 저장소입니다.

## 설치

```bash
uv sync
uv add mkdocs
```

## 문서 보기

```bash
# 개발 서버 실행
mkdocs serve

# 빌드
mkdocs build
```

문서는 `http://localhost:8000`에서 확인할 수 있습니다.

## 저장소 구조

```
os-make/
├── docs/          # 문서 소스 파일
├── mkdocs.yml     # MkDocs 설정 파일
└── README.md      # 이 파일
```

## 라이선스

이 문서는 교육 목적으로 작성되었습니다.

