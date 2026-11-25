# Cosmic Whispers 캐릭터 대화 엔진

**별의 속삭임**은 사주·타로를 소재로 한 한국어 비주얼 노벨입니다. 모델이 반환한 감정 태그로 캐릭터 표정을 고르고, 호감도 변화량으로 다음 대화의 관계 단계를 결정합니다. 이 사례는 Cosmic Whispers Team의 원본 구현을 기준 커밋에서 살펴봅니다. [원본 저장소](https://github.com/hyeongjun1291/cosmic-whispers) · [기준 커밋 `c99dd87`](https://github.com/hyeongjun1291/cosmic-whispers/tree/c99dd871da1065f1b0f68d9befc517ccafe25fec)

## 한 턴의 동작

1. **프롬프트 구성.** TypeScript로 정의한 네 캐릭터 중 하나의 페르소나를 고르고 누적 점수에 맞는 관계 단계(`30 미만`, `30–69`, `70 이상`)를 적용합니다. 청산은 사주 정보, 시온은 선택한 타로 카드 정보를 맥락으로 붙입니다. 기준 커밋의 서버는 이 맥락과 이전 메시지 최대 15개, 현재 입력을 Groq의 Llama 4 Scout에 보냅니다. 이 모델은 2026년 7월 17일 Groq 일반 요금제에서 종료됐습니다. [캐릭터 정의](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/lib/characters.ts) · [맥락 조립](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/app/page.tsx#L278-L306) · [Groq 요청](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/lib/groq.ts#L139-L161) · [Groq 모델 종료 공지](https://console.groq.com/docs/deprecations)
2. **응답 파싱.** 모델에 JSON 응답을 요청하고, 브라우저에 대사·감정·호감도 변화량을 전달합니다. JSON 파싱에 실패하면 생성 원문을 대사로 사용합니다. Groq가 JSON 검증 오류와 함께 실패한 생성 결과를 주고 그 안에서 `text`를 정규식으로 추출할 수 있으면, 해당 대사를 복구합니다. 화면은 감정 태그를 캐릭터 표정에 연결합니다. [응답 파서](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/lib/groq.ts#L163-L205) · [API 응답](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/app/api/chat/route.ts#L42-L57) · [표정 표시](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/components/VisualNovel.tsx#L75-L90)
3. **상태 갱신.** 브라우저는 캐릭터별 호감도를 `0–100` 범위로 유지합니다. 변화량의 절댓값이 2 이상이면 해당 턴을 브라우저 저장소에 기록합니다. 최근 기록은 최대 20개를 보관하고, 다음 요청에 최대 3개를 넣습니다. [대화 상태 갱신](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/app/page.tsx#L318-L338) · [기억 저장 및 선택](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/lib/memory.ts#L13-L49)

다음은 원본에서 응답의 변화량을 캐릭터별 누적 점수로 바꾸는 부분입니다. 예를 들어 기존 점수가 `68`이고 응답 변화량이 `+3`이면, 다음 요청의 점수는 `71`이고 프롬프트는 친밀 단계 설명을 사용합니다. [상태 갱신](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/app/page.tsx#L329-L336) · [프롬프트 단계 선택](https://github.com/hyeongjun1291/cosmic-whispers/blob/c99dd871da1065f1b0f68d9befc517ccafe25fec/src/lib/groq.ts#L33-L45)

```ts
const currentScore = affectionScores[characterId] || 30; // Default starts at 30
const delta = response.affectionScore;
const newScore = Math.min(100, Math.max(0, currentScore + delta));

const updatedScores = { ...affectionScores, [characterId]: newScore };
setAffectionScores(updatedScores);
localStorage.setItem('cosmic_affection', JSON.stringify(updatedScores));
```

후속 수정에서 모델 응답의 감정과 호감도 변화량을 검증하도록 보완했습니다. [후속 수정 `e60a103`](https://github.com/hyeongjun1291/cosmic-whispers/commit/e60a10347eb4b4dae563cb795ada9639b92f751f)
