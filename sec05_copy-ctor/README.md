# 05. 복사 생성자와 임시 객체

- 학습 문서: [C++ 강의 5섹션 – 복사 생성자와 임시 객체](https://claude.ai/artifact/B12vuXQX7aMQiGTmfWeHvr)
- 강의 범위: 슬라이드 66~83, 강의 예제 `05_*` (강의 예제는 `_local/`에만 보관)

## 과제

| 폴더 | 과제 | 난이도 | 제출할 파일 |
| --- | --- | --- | --- |
| `hw1_tracer/` | 호출 순서 예측하기 | 기본 | `main.cpp`, `NOTES.md` (실행 전 예측 vs 실제 출력) |
| `hw2_intarray/` | 깊은 복사하는 `CIntArray` | 기본 | `CIntArray.h`, `CIntArray.cpp`, `main.cpp` |
| `hw3_intarray-move/` | `CIntArray`에 이동 추가, 할당 횟수 측정 | 응용 | 위와 같음 + `NOTES.md` (빌드 옵션별 alloc 결과) |
| `hw4_cmystring-review/` | 강의 예제 `CMyString` 버그 찾기 | 도전 | 직접 작성한 테스트 `main.cpp`, `NOTES.md` (버그 설명과 수정 방법) |

## 메모

- `hw3`은 `hw2` 폴더를 복사한 상태로 먼저 커밋하고 시작합니다. 그러면 `git diff`로 추가한 부분만 볼 수 있습니다.
- `hw4`의 강의 원본 `CMyString` 코드는 올리지 않습니다. 수정 내용은 `NOTES.md`에 설명으로 적습니다.
- 각 폴더의 `.gitkeep`은 빈 폴더를 git에 올리기 위한 빈 파일입니다. 과제 파일을 넣으면 지워도 됩니다.
