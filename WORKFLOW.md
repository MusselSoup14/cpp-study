# 작업 순서

명령어는 MSYS2 터미널(또는 Git Bash) 기준입니다. 현재 **1단계**(main에서 add, commit, push) 기준으로 적었습니다.

## 0. 처음 한 번: 저장소 만들기

1. git 사용자 정보 설정 (PC마다 한 번)

    ```bash
    git config --global user.name "이름"
    git config --global user.email "GitHub 계정 이메일"
    ```

2. GitHub에서 공개(Public) 저장소 `cpp-study` 생성. README, .gitignore 추가 옵션은 **체크하지 않습니다**(이미 있음).
3. 받은 뼈대 폴더에서 첫 커밋

    ```bash
    cd cpp-study
    git init
    git add .
    git status                  # .gitignore, README.md, WORKFLOW.md, sec05_copy-ctor/... 가 보이는지 확인
    git commit -m "[docs] 저장소 뼈대 추가"
    ```

4. 원격 저장소 연결 후 push

    ```bash
    git branch -M main          # 기본 브랜치 이름을 GitHub과 같은 main으로 맞춤
    git remote add origin https://github.com/<계정>/cpp-study.git
    git remote -v               # origin이 등록됐는지 확인
    git push -u origin main
    ```

5. GitHub 페이지에서 파일이 올라갔는지 확인

## 1. 새 섹션 시작할 때

1. 강의자료로 섹션 학습 문서 요청
2. 섹션 폴더와 과제 폴더 만들기: `secNN_주제/README.md`, `hwN_이름/`
3. 루트 `README.md` 진도표에 섹션 한 줄 추가
4. 커밋, push

    ```bash
    git add .
    git commit -m "[secNN] 섹션 폴더 추가"
    git push
    ```

## 2. 과제 하나 할 때 (매번 반복)

1. 최신 상태 받기 (다른 PC에서 작업했을 수 있으니 항상 먼저)

    ```bash
    git pull
    ```

2. 과제 폴더에서 코드 작성, 빌드, 실행

    ```bash
    cd sec05_copy-ctor/hw2_intarray
    g++ -std=c++17 -Wall -Wextra *.cpp -o main && ./main
    ```

3. 의미 있는 단계마다 커밋 (빌드 통과, 기능 하나 완성)

    ```bash
    git status                  # 무엇이 바뀌었는지, .exe가 섞여 있지 않은지 확인
    git diff                    # 바뀐 내용 확인
    git add .
    git commit -m "[sec05/hw2] 복사 생성자 깊은 복사 구현"
    ```

4. 과제를 마치면 push

    ```bash
    git push
    ```

5. Claude에게 리뷰 요청: "sec05 hw2 리뷰해줘" + 저장소 링크
6. 리뷰 반영 후 커밋, push

    ```bash
    git add .
    git commit -m "[sec05/hw2] 리뷰 반영: 자기 대입 처리 추가"
    git push
    ```

7. 루트 `README.md` 진도표에서 `[ ]`를 `[x]`로 바꾸고 커밋, push

    ```bash
    git commit -am "[docs] 진도표 갱신: sec05 hw2 완료"
    git push
    ```

이전 과제를 확장하는 과제(예: hw3)는 이전 폴더를 복사해서 먼저 커밋한 뒤 시작합니다.

```bash
cp -r hw2_intarray/* hw3_intarray-move/
git add .
git commit -m "[sec05/hw3] hw2 코드로 시작"
```

## 3. 다른 PC에서 이어서 할 때

```bash
# 처음 한 번
git clone https://github.com/<계정>/cpp-study.git

# 이후에는 작업 전에 항상
git pull
```

작업을 마치면 그 PC에서 push까지 해둡니다. push하지 않으면 다른 PC에서 받을 수 없습니다.

## 4. 자주 만나는 상황

| 상황 | 해결 |
| --- | --- |
| push가 `rejected`됨 | 원격에 내가 없는 커밋이 있는 상태. `git pull`로 받은 뒤 다시 `git push` |
| pull 중 conflict 발생 | 파일에서 `<<<<<<<`, `=======`, `>>>>>>>` 사이를 원하는 내용으로 고치고, 표시 줄은 지운 뒤 `git add .` → `git commit` → `git push` |
| 커밋 메시지를 잘못 썼거나 파일을 빠뜨림 (push 전) | `git reset --soft HEAD^` 후 다시 add, commit |
| `.exe` 같은 파일을 실수로 add함 (커밋 전) | `git rm --cached <파일>` |
| 무엇이 바뀌었는지 모르겠음 | `git status`, `git diff`, `git log --oneline` |

push한 커밋은 `reset`으로 되돌리지 않습니다. 이미 원격에 올라간 이력이 꼬입니다. 고칠 내용은 새 커밋으로 추가합니다.

## 다음 단계에서 바뀌는 것

- **2단계**: 2번 과정을 과제마다 브랜치(`sec05-hw2`)에서 하고, 리뷰를 통과하면 main에 merge한 뒤 push합니다.
- **3단계**: 브랜치를 push하고 GitHub에서 PR을 열어 PR에서 리뷰를 받습니다.

단계가 바뀌면 이 문서와 README의 "현재 git 단계"를 함께 고칩니다.
