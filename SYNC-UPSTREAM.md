# Upstream Sync Guide (omc-slim)

이 문서는 oh-my-claudecode 원본(upstream)의 최신 변경사항을 가져온 뒤, openclaw 관련 코드를 다시 제거하는 절차를 설명합니다.

## 1. 수동 준비 (터미널에서 실행)

```bash
# upstream remote 등록 (최초 1회)
git remote add upstream https://github.com/Yeachan-Heo/oh-my-claudecode

# 원본 최신 가져오기
git fetch upstream

# merge (충돌 발생 시 수동 해결 후 commit)
git merge upstream/main
```

## 2. Claude에게 요청할 프롬프트

merge 완료 후 Claude Code에서 아래 프롬프트를 그대로 붙여넣으세요:

---

```
이 레포는 oh-my-claudecode를 fork한 slim 버전이야.
회사 보안 정책으로 "openclaw"이라는 텍스트가 코드베이스에 하나라도 있으면 git clone이 안 돼.

방금 upstream(원본)을 merge했는데, openclaw 관련 코드가 다시 들어왔을 수 있어.
아래 순서로 처리해줘:

1. grep -ri "openclaw" 으로 전체 코드베이스에서 openclaw 참조를 찾아줘
2. 찾은 것들을 모두 제거해줘 (기능은 유지하되 openclaw 관련 로직만 제거)
3. npm run build 로 dist/와 bridge/ 재빌드
4. tsc --noEmit 으로 타입 체크
5. vitest run 으로 관련 테스트 통과 확인
6. grep -ri "openclaw" 으로 최종 확인 (0건이어야 함)
7. 커밋해줘
```

---

## 3. 충돌이 발생하는 경우

merge 시 충돌이 나면 주로 아래 파일들입니다:

| 파일 | 이유 |
|------|------|
| `src/hooks/bridge.ts` | `_openclaw` 객체 영역 |
| `src/cli/launch.ts` | `--openclaw` 플래그 영역 |
| `src/notifications/config.ts` | legacy openclaw 마이그레이션 영역 |
| `src/notifications/presets.ts` | openclaw 프리셋 영역 |
| `src/notifications/types.ts` | replyChannel/replyTarget/replyThread 필드 |
| `README.*.md` | OpenClaw 통합 섹션 |

충돌 해결 시 **openclaw 관련 코드는 모두 제거하는 쪽(ours)**으로 해결하면 됩니다.

## 4. 참고사항

- 원본의 대부분 업데이트(agents, skills, hooks 개선)는 openclaw과 무관하므로 충돌 없이 merge됩니다
- `dist/`와 `bridge/`는 빌드 산출물이므로 소스(`src/`) 정리 후 `npm run build`로 재생성하면 됩니다
- 최초 openclaw 제거 커밋: `a955ec81`
