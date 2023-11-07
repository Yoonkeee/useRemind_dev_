# useRemind_dev_

```typescript
const THIS_REPO_IS_FOR = useRemind('dev')

type 회고 = 생각의 기록
type 학습 = 학습 내용의 기록
type 로그 = 개발 내용 기록

interface Issue {
  issue?: 'To-Do' | 'Working-On'
}

const SemanticCommitMessages = {
  types: ['학습', '개발', '생각', '기타']
}
```
