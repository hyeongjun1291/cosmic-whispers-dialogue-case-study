# Cosmic Whispers의 캐릭터 대화

**별의 속삭임**은 사주와 타로를 소재로 한 한국어 비주얼 노벨입니다. 대사와 함께 감정 태그가 오면 그에 맞는 표정을 띄웁니다. 호감도가 바뀌면 다음 요청에 새 점수를 보내고, 서버는 그 점수에 맞는 말투 지침을 프롬프트에 넣습니다.

Cosmic Whispers Team 프로젝트 · 코드 링크는 원본의 `c99dd87` 기준 [원본 저장소](https://github.com/hyeongjun1291/cosmic-whispers) · [기준 커밋 `c99dd87`](https://github.com/hyeongjun1291/cosmic-whispers/tree/c99dd871da1065f1b0f68d9befc517ccafe25fec)

## 모델에 보내는 정보

브라우저는 현재 캐릭터의 누적 호감도와 대화 이력을 `/api/chat`에 보냅니다. 청산과 대화할 때는 사주 요약을, 시온과 대화할 때는 선택한 타로 카드의 이름과 의미를 함께 보냅니다. 이전에 따로 저장한 대화가 있으면 요청에 덧붙입니다. [브라우저에서 요청 조립](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/app/page.tsx#L278-L306)

서버는 네 캐릭터 중 하나의 페르소나와 현재 관계 단계(`30 미만`, `30–69`, `70 이상`)로 시스템 프롬프트를 구성합니다. 이전 메시지 최대 15개와 새 메시지를 덧붙여 Groq에 JSON 응답을 요청합니다. [캐릭터 정의](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/lib/characters.ts) · [관계 단계 선택](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/lib/groq.ts#L33-L45) · [모델 요청](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/lib/groq.ts#L139-L161)

## 시온의 호감도가 68에서 71이 될 때

시온의 호감도가 `68`일 때 API가 아래처럼 응답했다고 가정합니다. 대화에 쓰이는 세 필드만 적었습니다. [API 반환 코드](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/app/api/chat/route.ts#L52-L57)

```json
{
  "text": "(그가 카드를 한 장 뒤집었다) 먼저 지금 할 수 있는 일부터 정해 보죠.",
  "emotion": "smile",
  "affectionScore": 3
}
```

| 응답 필드 | 브라우저에서 하는 일 |
| --- | --- |
| `text` | 캐릭터의 대사를 화면에 표시하고 대화 이력에 추가합니다. |
| `emotion` | `smile`에 해당하는 캐릭터 표정 이미지를 고릅니다. |
| `affectionScore` | 변화량 `+3`을 기존 점수에 더해 `71`을 저장합니다. 앱이 누적 점수를 `0–100`으로 제한합니다. |

응답의 `affectionScore`는 **이번 턴의 변화량**이고, 다음 요청의 `affectionScore`는 **갱신된 누적 점수**입니다. 다음 요청에는 `71`이 들어가므로 서버가 친밀 단계의 말투 지침을 넣습니다. 이 턴은 변화량이 `+3`이라 기억으로도 저장합니다. 기준은 변화량의 절댓값 2 이상입니다. 브라우저는 최근 기억을 최대 20개 보관하고, 한 요청에 최대 3개를 넣습니다. [표정 선택](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/components/VisualNovel.tsx#L75-L90) · [상태 갱신](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/app/page.tsx#L318-L338) · [기억 저장과 선택](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/lib/memory.ts#L13-L49)

## 응답을 읽지 못했을 때

원본 파서는 JSON 파싱에 실패하면 생성 원문을 대사로 쓰고 기본 표정을 적용합니다. Groq가 JSON 검증 오류와 함께 실패한 생성 결과를 보내면, 그 안에서 `text`를 추출할 수 있을 때 대사를 복구합니다. [응답 파서와 복구 코드](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/lib/groq.ts#L163-L205)

이후 감정 태그와 호감도 변화량을 검사하는 코드를 추가했습니다. [후속 수정 `e60a103`](https://github.com/hyeongjun1291/cosmic-whispers/commit/e60a10347eb4b4dae563cb795ada9639b92f751f)
