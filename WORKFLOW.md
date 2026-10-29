# 작업 순서

명령어는 MSYS2 터미널(또는 Git Bash) 기준입니다. 현재 **1단계**(main에서 add, commit, push) 기준으로 적었습니다.

## 0. 처음 한 번: 저장소 만들기

로컬 폴더에 이미 있는 파일로 저장소를 만들고 GitHub에 올리는 순서입니다(로컬 먼저 방식). 저장소를 처음 만들 때 한 번만 합니다. 이미 GitHub에 있는 저장소를 다른 PC에서 쓸 때는 "3. 다른 PC에서 이어서 할 때"를 따릅니다.

### 0-1. git 사용자 정보 확인

```bash
git config --global user.name
git config --global user.email
```

- **하는 일**: 커밋에 기록될 작성자 이름과 이메일을 보여줍니다.
- **왜**: 모든 커밋에는 "누가 만들었는지"가 기록됩니다. 비어 있으면 커밋이 거부됩니다.
- 아무것도 출력되지 않으면 설정합니다(PC마다 한 번).

    ```bash
    git config --global user.name "이름"
    git config --global user.email "GitHub 계정 이메일"
    ```

- GitHub 계정 이메일과 같아야 GitHub에서 커밋이 내 계정과 연결되어 보입니다.

### 0-2. 로컬 저장소 만들기

`cpp-study` 폴더로 이동한 뒤 실행합니다. VS Code로 `cpp-study`를 열었다면 VS Code 터미널은 이미 이 폴더에서 열립니다.

```bash
git init
```

- **하는 일**: 현재 폴더를 git 저장소로 만듭니다. 숨김 폴더 `.git`이 생기고, 앞으로의 버전 기록이 모두 여기에 저장됩니다.
- **왜**: 이 명령 전에는 그냥 일반 폴더라 git 명령이 동작하지 않습니다.
- `.git` 폴더는 지우거나 직접 고치지 않습니다. 지우면 버전 기록이 모두 사라집니다.

### 0-3. 첫 커밋할 파일 올리기 (staging)

```bash
git add .
```

- **하는 일**: 현재 폴더의 모든 변경 사항을 staging area(커밋 후보)에 올립니다. `.`은 "현재 폴더 전체"라는 뜻입니다.
- `.gitignore`에 적힌 파일(`.exe`, `_local/` 등)은 자동으로 빠집니다.
- 이때 아래와 같은 경고가 나올 수 있습니다. **무시해도 됩니다.**

    ```text
    warning: in the working copy of 'README.md', LF will be replaced by CRLF the next time Git touches it
    ```

    줄 끝 표시 방식이 리눅스(LF)와 Windows(CRLF)가 다르기 때문에 나오는 안내입니다. Git for Windows는 저장소에는 LF로 저장하고, PC에 파일을 꺼낼 때 CRLF로 바꿔줍니다. 빌드나 편집에는 영향이 없습니다.

### 0-4. 올라간 파일 확인

```bash
git status
```

- **하는 일**: 지금 staging area에 무엇이 올라갔는지(`new file: ...`) 보여줍니다.
- **왜**: 커밋하기 전에 넣으면 안 되는 파일(`.exe`, 테스트용 파일, 강의자료)이 섞이지 않았는지 확인합니다.
- **반드시 `git add` 다음에 확인합니다.** `add` 전에 보면 새 폴더는 `.vscode/`처럼 폴더 이름만 보여서 안의 파일을 확인할 수 없습니다.
- 넣으면 안 되는 파일이 보이면: 그 파일을 지우거나 옮긴 뒤 `git add .`를 다시 실행하고 다시 확인합니다. `git add .`는 삭제한 파일도 반영합니다.

### 0-5. 첫 커밋

```bash
git commit -m "[docs] 저장소 뼈대 추가"
git log --oneline
```

- **하는 일**: staging area의 내용을 하나의 버전(커밋)으로 로컬 저장소에 기록합니다. `-m` 뒤가 커밋 메시지입니다.
- `git log --oneline`: 커밋 목록을 한 줄씩 보여줍니다. 방금 만든 커밋이 보이면 로컬 작업은 끝입니다.
- 커밋은 로컬에서만 일어나는 일이라 원격 저장소가 없어도 됩니다.

### 0-6. GitHub 저장소 만들기

1. GitHub → **New repository**
2. Repository name: `cpp-study`, **Public** 선택
3. **Add a README file, Add .gitignore, Choose a license는 모두 선택하지 않습니다.**
4. **Create repository** → 다음 화면에 나오는 저장소 URL(`https://github.com/<계정>/cpp-study.git`)을 복사

3번이 중요합니다. 옵션을 선택하면 GitHub 쪽에 첫 커밋이 따로 생깁니다. 그러면 로컬의 첫 커밋과 이력이 서로 달라져서 push가 거부(rejected)되고, 두 이력을 합치는 추가 작업이 필요해집니다. 빈 저장소로 만들면 로컬 커밋이 그대로 올라갑니다.

### 0-7. 기본 브랜치 이름 맞추기

```bash
git branch -M main
```

- **하는 일**: 현재 브랜치의 이름을 `main`으로 바꿉니다. `-M`은 같은 이름이 있어도 강제로 바꾸는 옵션입니다.
- **왜**: Windows의 git은 설치 설정에 따라 첫 브랜치를 `master`로 만듭니다. GitHub의 기본 브랜치 이름은 `main`이라 이름을 맞춰둬야 헷갈리지 않습니다.
- 이미 `main`이라면 실행해도 아무 변화가 없습니다. `git branch`로 현재 브랜치 이름을 볼 수 있습니다(`*` 표시가 현재 브랜치).

### 0-8. 원격 저장소 연결

```bash
git remote add origin https://github.com/<계정>/cpp-study.git
```

- **하는 일**: 이 로컬 저장소에 원격 저장소 주소를 등록합니다. `origin`은 그 주소에 붙이는 **별명**입니다.
- **왜**: 이후 push, pull 할 때 긴 URL 대신 `origin`이라는 이름으로 부르기 위해서입니다. `origin`은 관례적인 이름일 뿐 다른 이름도 쓸 수 있습니다.
- 이 명령은 연결 정보만 저장합니다. 아직 아무것도 올라가지 않습니다.

```bash
git remote -v
```

- **하는 일**: 등록된 원격 저장소 목록을 보여줍니다. `-v`는 별명과 URL을 같이 보여주는 옵션입니다.
- 아래처럼 두 줄이 나오면 정상입니다(가져오기용 fetch, 올리기용 push).

    ```text
    origin  https://github.com/<계정>/cpp-study.git (fetch)
    origin  https://github.com/<계정>/cpp-study.git (push)
    ```

- URL을 잘못 넣었다면: `git remote set-url origin <올바른 URL>`

### 0-9. 처음 push

```bash
git push -u origin main
```

- **하는 일**: 로컬의 `main` 브랜치 커밋을 `origin`(GitHub)의 `main` 브랜치로 올립니다.
- **`-u`의 의미**: 로컬 `main`과 `origin`의 `main`을 짝으로 기억시킵니다(upstream 설정). 처음 한 번만 붙이면, 이후에는 `git push`, `git pull`만 쳐도 자동으로 `origin main`과 주고받습니다.
- 처음 push할 때 GitHub 로그인 창이 뜰 수 있습니다. 브라우저로 로그인하면 인증 정보가 Windows에 저장되어 다음부터는 묻지 않습니다.

### 0-10. 확인

- GitHub 저장소 페이지를 새로고침해서 파일이 보이는지 확인합니다.
- `.exe`, `_local/`, `.vscode`의 다른 파일(`settings.json` 등)이 올라가지 않았는지 확인합니다.

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

새 PC라면 [SETUP.md](SETUP.md)의 "다른 PC에서 clone해서 쓰는 경우" 순서로 개발 환경부터 준비합니다.

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
