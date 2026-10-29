# 개발 환경 설정 (Windows + MSYS2 UCRT64 + VS Code)

C++ 실습용 개발 환경을 구성하는 순서입니다. 각 단계마다 **왜 하는지**를 먼저 적고 방법을 적었습니다.

- 새 PC에 처음 설치한다면: 1 → 2 → 4 → 5 → 6 → 7 → 8
- 이미 다른 MinGW가 설치된 PC라면: 1 → 3 → 4 → 5 → 6 → 7 → 8

---

## 1. 구성 요소와 설치하는 이유

| 구성 요소 | 하는 일 | 왜 필요한가 |
| --- | --- | --- |
| GCC (`gcc`, `g++`) | C/C++ 컴파일러, 링커 | 무료이고 C++ 표준을 잘 따르는 대표 컴파일러. STM32CubeIDE의 `arm-none-eabi-gcc`, 임베디드 리눅스도 같은 GCC 계열이라 옵션과 에러 메시지가 그대로 이어진다 |
| MinGW-w64 | Windows용 GCC | GCC는 원래 리눅스/유닉스용이다. Windows `.exe`를 만들려면 Windows 헤더와 런타임에 맞춘 버전이 필요하다 |
| MSYS2 | MinGW-w64 설치·업데이트 관리자 + bash 터미널 | `pacman` 명령 한 줄로 설치하고 `pacman -Syu`로 한 번에 업데이트한다. 직접 받아 설치하는 것보다 관리가 쉽다 |
| UCRT64 환경 | MSYS2 안의 개발 환경 중 하나 | Windows 10 이상의 표준 C 런타임(UCRT)을 쓴다. 현재 MSYS2와 VS Code 공식 문서의 권장 환경이다 |
| GDB | 디버거 | 중단점, 한 줄씩 실행, 변수 확인 |
| VS Code | 편집기 | 자체 컴파일러는 없다. 설치한 GCC와 GDB를 불러서 쓴다 |
| C/C++ 확장 | VS Code의 C++ 기능 | 자동완성과 오류 표시(IntelliSense), GDB 연동 디버깅 |
| Git | 버전 관리 | 과제 저장소 관리 |

### MSYS2 환경이란

MSYS2를 설치하면 그 안에 서로 독립된 개발 환경이 여러 개 들어온다. 환경마다 설치 폴더, 터미널, 패키지 이름이 따로 있다.

| 환경 | 설치 폴더 | 패키지 이름 접두어 | 용도 |
| --- | --- | --- | --- |
| **UCRT64** | `C:\msys64\ucrt64` | `mingw-w64-ucrt-x86_64-` | **이 저장소에서 사용** |
| MINGW64 | `C:\msys64\mingw64` | `mingw-w64-x86_64-` | 예전 권장 환경. 현재는 legacy로 분류 |
| MSYS | `C:\msys64\usr` | 접두어 없음 | `pacman`, bash 등 MSYS2 자체 도구용. **개발용 아님** |

주의할 점

- 개발용 GCC는 반드시 `mingw-w64-ucrt-x86_64-` 접두어가 붙은 패키지로 설치한다. 접두어 없이 `pacman -S gcc`를 하면 MSYS 환경용이 설치되고, 그걸로 만든 `.exe`는 `msys-2.0.dll` 없이는 실행되지 않는다.
- 환경을 섞어 쓰지 않는다. PATH에도 한 환경의 `bin` 폴더 하나만 넣는다.

---

## 2. MSYS2와 UCRT64 툴체인 설치 (새 PC)

### 2-1. MSYS2 설치

- **왜**: GCC를 설치하고 업데이트할 관리자를 먼저 설치한다.
- [msys2.org](https://www.msys2.org)에서 설치 프로그램을 받아 실행한다.
- 설치 폴더는 기본값 `C:\msys64`를 그대로 쓴다. 경로에 공백이나 한글이 없어야 문제가 적다.

### 2-2. MSYS2 업데이트

- **왜**: 설치 프로그램에 들어 있는 패키지는 최신이 아닐 수 있다.
- 시작 메뉴에서 **MSYS2 UCRT64** 터미널을 열고 실행한다.

    ```bash
    pacman -Syu
    ```

- 터미널이 스스로 닫히면 다시 열어서 한 번 더 실행한다. "there is nothing to do"가 나오면 끝.

### 2-3. 툴체인 설치

- **왜**: 컴파일러(`gcc`, `g++`), 디버거(`gdb`), 링커, `make`를 한 번에 설치한다.

    ```bash
    pacman -S --needed base-devel mingw-w64-ucrt-x86_64-toolchain
    ```

- 패키지 선택 질문이 나오면 Enter(전체 선택), 진행 여부는 `Y`.
- 설치 결과는 `C:\msys64\ucrt64\bin`에 들어간다.

### 2-4. PATH 등록

- **왜**: VS Code와 명령 프롬프트가 `g++.exe`, `gdb.exe`를 어디서 찾을지 Windows에 알려준다.
- 시작 메뉴에서 **"계정의 환경 변수 편집"** → 사용자 변수 `Path` → 편집 → 새로 만들기 → `C:\msys64\ucrt64\bin` 추가 → 확인.
- 열려 있던 명령 프롬프트와 VS Code는 모두 닫았다가 다시 연다. 이미 열려 있던 창에는 바뀐 PATH가 적용되지 않는다.

---

## 3. 이미 다른 MinGW가 설치된 경우 (PATH 정리)

- **왜**: `g++`가 여러 개 설치되어 있으면 PATH에서 먼저 찾은 것이 실행된다. 의도하지 않은 컴파일러로 빌드되면 런타임이 섞여서 링크 오류나 실행 오류가 날 수 있다.

### 3-1. 현재 상태 확인

명령 프롬프트에서 실행한다.

```bat
where gcc
where g++
where gdb
```

`where`는 찾은 순서대로 보여주며, **맨 위 줄이 실제로 실행되는 것**이다.

예시 (이 PC의 정리 전 상태)

```text
C:\Program Files\mingw64\bin\gcc.exe    ← 별도 설치한 MinGW. 현재 이것이 실행됨
C:\msys64\ucrt64\bin\gcc.exe            ← MSYS2 UCRT64. 뒤로 밀려 있음
```

Windows는 PATH를 **시스템 변수 → 사용자 변수** 순서로 찾는다. `Program Files`에 설치된 프로그램은 보통 시스템 변수에 등록되기 때문에, 사용자 변수에 넣은 UCRT64보다 앞에 오게 된다.

### 3-2. 기존 MinGW를 PATH에서 빼기

1. 시작 메뉴에서 **"시스템 환경 변수 편집"** → 환경 변수 (관리자 권한 필요)
2. **시스템 변수**의 `Path`에서 `C:\Program Files\mingw64\bin`을 삭제
3. **사용자 변수**의 `Path`에도 있으면 삭제
4. 사용자 변수 `Path`에 `C:\msys64\ucrt64\bin`이 있는지 확인. 없으면 2-4대로 추가
5. 명령 프롬프트와 VS Code를 모두 닫았다가 다시 연다

Qt에 딸린 MinGW(`C:\Qt\Tools\mingw...\bin`)가 PATH에 있다면 그것도 빼는 것을 권장한다. Qt Creator는 PATH와 상관없이 자기 컴파일러를 찾아 쓰므로 Qt 작업에는 영향이 없다.

### 3-3. 기존 MinGW 폴더 정리 (나중에)

- PATH에서 뺀 뒤 며칠 써보고 문제가 없으면 지운다.
- 설치 프로그램으로 설치했다면 "앱 및 기능"에서 제거, 압축을 풀어 넣었다면 폴더 삭제.
- 지우기 전에 예전 프로젝트의 빌드 스크립트나 IDE 설정이 그 경로를 직접 쓰고 있지 않은지 확인한다.

### 3-4. 기존 설정과 실행 파일 정리

- 기존 `.vscode` 폴더의 설정 파일에 `C:\Program Files\mingw64\...` 경로가 적혀 있으면 지우거나 6장대로 다시 만든다.
- 예전에 빌드한 `.exe`는 다시 빌드한다. 예전 런타임 DLL을 찾다가 실행되지 않을 수 있다.

---

## 4. 설치 확인

명령 프롬프트를 **새로 열고** 실행한다.

```bat
where gcc
where g++
where gdb
g++ --version
gdb --version
```

- `where` 세 개 모두 `C:\msys64\ucrt64\bin\...` **한 줄만** 나오면 정상.
- `g++ --version` 첫 줄에 `Built by MSYS2 project`가 보이면 MSYS2 컴파일러다.
- `gdb`를 찾을 수 없다면 MSYS2 UCRT64 터미널에서 설치한다.

    ```bash
    pacman -S --needed mingw-w64-ucrt-x86_64-gdb
    ```

---

## 5. VS Code와 C/C++ 확장 설치

- **왜**: VS Code는 편집기이고, C++ 지원은 확장이 붙여준다.
1. [code.visualstudio.com](https://code.visualstudio.com)에서 설치
2. 확장(Extensions, `Ctrl+Shift+X`)에서 **C/C++** (게시자: Microsoft, ID `ms-vscode.cpptools`) 설치
3. **IntelliSense 컴파일러 지정**: `Ctrl+Shift+P` → `C/C++: Select IntelliSense Configuration` → `C:\msys64\ucrt64\bin\g++.exe` 선택
    - **왜**: 편집기의 자동완성과 빨간 줄 표시가 실제 빌드와 같은 컴파일러 기준으로 동작하게 한다. 빌드 자체에는 영향이 없다.

---

## 6. 빌드·디버그 설정 직접 만들기 (`tasks.json`, `launch.json`)

`.vscode` 폴더의 두 파일이 VS Code의 빌드와 디버그 방법을 정한다.

| 파일 | 역할 |
| --- | --- |
| `tasks.json` | 빌드 명령. `Ctrl+Shift+B`로 실행 |
| `launch.json` | 디버거 실행 방법. `F5`로 실행 |
| `c_cpp_properties.json` | 편집기 코드 분석(IntelliSense) 설정. 5장 3번에서 생성될 수 있으며 저장소에 올리지 않는다 |

저장소 폴더 `cpp-study` **자체**를 VS Code로 연다(파일 → 폴더 열기). 그래야 `.vscode`가 저장소 루트에 만들어진다.

### 6-1. tasks.json 만들기

1. 과제 폴더에 테스트용 `main.cpp`를 하나 만든다.

    ```cpp
    #include <iostream>
    int main() { std::cout << "hello" << std::endl; return 0; }
    ```

2. `main.cpp`를 연 상태에서 메뉴 **터미널 → 기본 빌드 작업 구성** → **C/C++: g++.exe 활성 파일 빌드** (`C:\msys64\ucrt64\bin\g++.exe`가 적힌 항목) 선택
3. `.vscode/tasks.json`이 생성된다. 생성된 내용을 아래 기준으로 고친다.

| 항목 | 생성된 기본값 | 고칠 값 | 이유 |
| --- | --- | --- | --- |
| `label` | `C/C++: g++.exe 활성 파일 빌드` 등 | `C++: build folder` | 짧고 알아보기 쉽게. `launch.json`에서 이 이름으로 부른다 |
| `command` | `C:\\msys64\\ucrt64\\bin\\g++.exe` | `g++` | PATH에서 찾게 해서 PC마다 경로가 달라도 동작 |
| `args`의 `${file}` | 열린 파일 하나만 빌드 | `${fileDirname}\\*.cpp` | 과제 폴더의 `.cpp`(예: `main.cpp`, `CIntArray.cpp`)를 모두 빌드 |
| `args`의 출력 이름 | `${fileDirname}\\${fileBasenameNoExtension}.exe` | `${fileDirname}\\main.exe` | 실행 파일 이름을 고정해서 `launch.json`이 찾기 쉽게 |
| `args`에 추가 | 없음 | `-std=c++17`, `-Wall`, `-Wextra` | C++ 표준 지정, 경고를 최대한 켜서 실수 발견 |

참고 예시 (직접 만든 뒤 비교용)

```jsonc
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "C++: build folder",
            "type": "cppbuild",
            "command": "g++",
            "args": [
                "-fdiagnostics-color=always",
                "-g",                              // 디버그 정보 포함 (GDB에 필요)
                "-std=c++17",
                "-Wall",
                "-Wextra",
                "${fileDirname}\\*.cpp",           // 현재 열린 파일이 있는 폴더의 모든 .cpp
                "-o",
                "${fileDirname}\\main.exe"
            ],
            "options": { "cwd": "${fileDirname}" },
            "problemMatcher": ["$gcc"],
            "group": { "kind": "build", "isDefault": true }
        }
    ]
}
```

4. `main.cpp`를 연 상태에서 `Ctrl+Shift+B` → 터미널에 오류 없이 끝나고 과제 폴더에 `main.exe`가 생기면 성공.

**(선택) 복사 생략 끄기 빌드**: 5섹션처럼 생성자 호출 순서를 원리대로 보고 싶을 때 쓴다. 위 작업을 복사해 `tasks` 배열에 하나 더 넣고, `label`을 `C++: build folder (no elide)`로, `-std=c++17`을 `-std=c++14`와 `-fno-elide-constructors` 두 줄로 바꾼다. `group`은 `"build"`로 둔다(기본 작업은 하나만). 실행은 `Ctrl+Shift+P` → `Tasks: Run Build Task`에서 선택.

### 6-2. launch.json 만들기

1. `main.cpp`를 연 상태에서 메뉴 **실행 → 구성 추가** → **C++ (GDB/LLDB)** 선택
2. `.vscode/launch.json`이 생성된다. 구성이 비어 있으면 오른쪽 아래 **구성 추가** 버튼 → **C/C++: (gdb) 시작**을 선택한다.
3. 아래 기준으로 고친다.

| 항목 | 고칠 값 | 이유 |
| --- | --- | --- |
| `name` | `C++: debug folder` | 실행 화면에 보이는 이름 |
| `program` | `${fileDirname}\\main.exe` | 6-1에서 만든 실행 파일 |
| `cwd` | `${fileDirname}` | 프로그램이 과제 폴더 기준으로 실행되게 |
| `miDebuggerPath` | `C:\\msys64\\ucrt64\\bin\\gdb.exe` | 사용할 디버거. MSYS2를 기본 경로에 설치하면 어느 PC든 같다 |
| `preLaunchTask` | `C++: build folder` | 디버깅 전에 자동으로 빌드. 6-1의 `label`과 **글자 하나까지 같아야** 한다 |
| `externalConsole` | `false` | 출력이 VS Code 안의 터미널에 나오게 |

참고 예시

```jsonc
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "C++: debug folder",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}\\main.exe",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "gdb",
            "miDebuggerPath": "C:\\msys64\\ucrt64\\bin\\gdb.exe",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C++: build folder"
        }
    ]
}
```

4. `main.cpp`의 `std::cout` 줄 왼쪽을 클릭해 중단점(빨간 점)을 찍고 `F5` → 빌드 후 그 줄에서 멈추면 성공. `F10`(다음 줄), `F11`(함수 안으로), `F5`(계속)로 진행해본다.

### 6-3. 확인이 끝나면

테스트용 `main.cpp`와 `main.exe`는 지운다. `main.exe`는 `.gitignore`의 `*.exe` 규칙 때문에 어차피 커밋되지 않는다.

---

## 7. `.gitignore` 수정: 두 설정 파일만 저장소에 올리기

- **왜**: `tasks.json`, `launch.json`을 저장소에 올리면 다른 PC에서 clone만 해도 같은 빌드·디버그 설정을 쓸 수 있다. 나머지 `.vscode` 파일(`settings.json`, `c_cpp_properties.json`)은 PC나 개인 취향에 따른 설정이라 올리지 않는다.

`.gitignore`에서

```gitignore
.vscode/
```

를 아래 세 줄로 바꾼다.

```gitignore
.vscode/*
!.vscode/tasks.json
!.vscode/launch.json
```

- `.vscode/*`: `.vscode` 안의 파일을 모두 제외
- `!`: 제외에서 다시 포함

`.vscode/`(폴더 자체 제외)로 두면 `!` 규칙이 동작하지 않는다. git은 제외된 폴더 안을 아예 들여다보지 않기 때문이다. 그래서 `.vscode/*`(폴더 안의 파일 제외)로 바꿔야 한다.

확인과 커밋

```bash
git status            # .gitignore, .vscode/tasks.json, .vscode/launch.json만 보이면 정상
git add .
git commit -m "[setup] VS Code 빌드·디버그 설정 추가"
git push
```

---

## 8. Git 설치와 첫 설정

- **왜**: 과제 저장소를 관리한다. VS Code의 소스 제어 화면도 이 Git을 사용한다.
- 이미 설치되어 있는지 먼저 확인한다.

    ```bat
    git --version
    ```

- 없으면 [git-scm.com](https://git-scm.com)에서 **Git for Windows**를 설치한다. 설치 옵션은 기본값으로 둔다. Git Bash 터미널도 함께 설치된다.
    - MSYS2에서 `pacman -S git`으로도 설치할 수 있지만, 그 git은 MSYS2 터미널 안에서만 동작하고 명령 프롬프트와 VS Code에서는 찾지 못한다. 그래서 Git for Windows를 권장한다.
- 사용자 정보 설정 (PC마다 한 번)

    ```bash
    git config --global user.name "이름"
    git config --global user.email "GitHub 계정 이메일"
    ```

이후 저장소 만들기는 [WORKFLOW.md](WORKFLOW.md)의 "0. 처음 한 번"을 따른다.

---

## 9. 문제 해결

| 증상 | 원인 | 해결 |
| --- | --- | --- |
| `'g++'은(는) 내부 또는 외부 명령...이 아닙니다` | PATH에 `C:\msys64\ucrt64\bin`이 없거나, 창을 다시 열지 않음 | 2-4 확인 후 명령 프롬프트, VS Code 모두 다시 열기 |
| PATH를 고쳤는데 `where g++` 맨 위가 여전히 다른 경로 | 시스템 변수 PATH에 다른 MinGW가 남아 있음 | 3-2대로 시스템 변수 확인 |
| 빌드는 되는데 편집기에 빨간 줄이 뜸 | IntelliSense가 다른 컴파일러나 다른 C++ 표준 기준으로 분석 중 | 5장 3번 다시 실행, `C/C++: Edit Configurations (UI)`에서 C++ 표준을 `c++17`로 |
| `*.cpp: No such file or directory` | `.cpp`가 아닌 파일(예: `README.md`)을 연 상태로 빌드 | 빌드할 과제 폴더의 `.cpp` 파일을 연 상태에서 `Ctrl+Shift+B` |
| 다른 과제 폴더가 빌드됨 | `${fileDirname}`은 **지금 열려 있는 파일**의 폴더 | 빌드할 과제의 파일을 클릭한 뒤 빌드 |
| `F5` 시 `preLaunchTask ... 을(를) 찾을 수 없습니다` | `launch.json`의 `preLaunchTask`와 `tasks.json`의 `label`이 다름 | 두 값을 똑같이 맞추기 |
| `main.exe: Permission denied` 또는 빌드 시 파일 쓰기 실패 | 이전에 실행한 `main.exe`가 아직 실행 중 | 실행 중인 프로그램을 종료(`Ctrl+C`)하거나 디버깅을 중지한 뒤 다시 빌드 |
| 출력의 한글이 깨짐 | 소스 파일은 UTF-8인데 콘솔은 CP949 | 실습 출력은 영문으로 쓰거나, 터미널에서 `chcp 65001` 실행 후 다시 실행 |
| `msys-2.0.dll이 없어` 실행되지 않음 | MSYS 환경 GCC(접두어 없는 `gcc` 패키지)로 빌드됨 | 1장 주의사항 확인, UCRT64 툴체인으로 다시 빌드 |

---

## 참고

- [Using GCC with MinGW – VS Code 공식 문서](https://code.visualstudio.com/docs/cpp/config-mingw)
- [MSYS2](https://www.msys2.org)
