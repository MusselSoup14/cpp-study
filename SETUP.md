# 개발 환경 설정 (Windows + MSYS2 UCRT64 + VS Code)

C++ 실습용 개발 환경을 구성하는 순서입니다. 각 단계마다 **왜 하는지**를 먼저 적고 방법을 적었습니다.

| 상황 | 순서 |
| --- | --- |
| 새 PC에서 저장소를 처음 만드는 경우 | 1 → 2 → 4 → 5 → 6-1 → 7 → 8 → 9 |
| 다른 PC에서 이 저장소를 clone해서 쓰는 경우 | 1 → 2 → 4 → 5 → 6-2 → 7 → 8(확인만) |
| 다른 MinGW가 이미 설치된 PC | 위 순서에서 2 대신 3 (MSYS2도 없다면 2와 3 모두) |

VS Code가 영어 화면이면 메뉴 이름은 괄호 안의 영어를 봅니다.

---

## 1. 구성 요소와 설치하는 이유

| 구성 요소 | 하는 일 | 왜 필요한가 |
| --- | --- | --- |
| GCC (`gcc`, `g++`) | C/C++ 컴파일러, 링커 | 무료이고 C++ 표준을 잘 따르는 대표 컴파일러. STM32CubeIDE의 `arm-none-eabi-gcc`, 임베디드 리눅스도 같은 GCC 계열이라 옵션과 에러 메시지가 그대로 이어진다 |
| MinGW-w64 | Windows용 GCC | GCC는 원래 리눅스/유닉스용이다. Windows `.exe`를 만들려면 Windows 헤더와 런타임에 맞춘 버전이 필요하다 |
| MSYS2 | MinGW-w64 설치·업데이트 관리자 + bash 터미널 | `pacman` 명령 한 줄로 설치하고 `pacman -Syu`로 한 번에 업데이트한다. 직접 받아 설치하는 것보다 관리가 쉽다 |
| UCRT64 환경 | MSYS2 안의 개발 환경 중 하나 | Windows 10 이상의 표준 C 런타임(UCRT)을 쓴다. 현재 MSYS2와 VS Code 공식 문서의 권장 환경이다 |
| GDB | 디버거 | 중단점, 한 줄씩 실행, 변수 확인 |
| Git | 버전 관리 | 과제 저장소 관리 |
| VS Code | 편집기 | 자체 컴파일러는 없다. 설치한 GCC와 GDB를 불러서 쓴다 |
| C/C++ 확장 | VS Code의 C++ 기능 | 자동완성과 오류 표시(IntelliSense), GDB 연동 디버깅. 확장을 설치해도 컴파일러는 들어 있지 않다 |

### MSYS2 환경이란

MSYS2를 설치하면 그 안에 서로 독립된 개발 환경이 여러 개 들어온다. 환경마다 설치 폴더, 터미널, 패키지 이름이 따로 있다.

| 환경 | 설치 폴더 | 패키지 이름 접두어 | 용도 |
| --- | --- | --- | --- |
| **UCRT64** | `C:\msys64\ucrt64` | `mingw-w64-ucrt-x86_64-` | **이 저장소에서 사용** |
| MINGW64 | `C:\msys64\mingw64` | `mingw-w64-x86_64-` | 예전 권장 환경. 현재는 legacy로 분류 |
| MSYS | `C:\msys64\usr` | 접두어 없음 | `pacman`, bash 등 MSYS2 자체 도구용. **개발용 아님** |

- 시작 메뉴의 "MSYS2 UCRT64", "MSYS2 MINGW64" 등은 같은 bash 터미널인데, 그 환경의 `bin` 폴더가 PATH 맨 앞에 오도록 설정만 다르다.
- 개발용 GCC는 반드시 `mingw-w64-ucrt-x86_64-` 접두어가 붙은 패키지로 설치한다. 접두어 없이 `pacman -S gcc`를 하면 MSYS 환경용이 설치되고, 그걸로 만든 `.exe`는 `msys-2.0.dll` 없이는 실행되지 않는다.
- 환경을 섞어 쓰지 않는다. PATH에도 한 환경의 `bin` 폴더 하나만 넣는다.

---

## 2. MSYS2와 UCRT64 툴체인 설치

### 2-1. MSYS2 설치

- **왜**: GCC를 설치하고 업데이트할 관리자를 먼저 설치한다.
- [msys2.org](https://www.msys2.org)에서 설치 프로그램을 받아 실행한다.
- 설치 폴더는 기본값 `C:\msys64`를 그대로 쓴다. 경로에 공백이나 한글이 없어야 문제가 적고, 저장소의 `launch.json`도 이 경로를 기준으로 되어 있다.

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
- 시스템 변수 `Path`에 이미 `C:\msys64\ucrt64\bin`이 있다면 추가하지 않는다. PATH는 시스템 변수와 사용자 변수를 합쳐서 쓰므로 **둘 중 한 곳에만 있으면 된다.**
- 열려 있던 명령 프롬프트와 VS Code는 모두 닫았다가 다시 연다. 이미 열려 있던 창에는 바뀐 PATH가 적용되지 않는다.

---

## 3. 이미 다른 MinGW가 설치된 경우 (PATH 정리)

- **왜**: `g++`가 여러 개 설치되어 있으면 PATH에서 먼저 찾은 것이 실행된다. 의도하지 않은 컴파일러로 빌드되면 런타임이 섞여서 링크 오류나 실행 오류가 날 수 있다.

### 3-1. 현재 상태 확인

**명령 프롬프트(cmd)**에서 실행한다.

```bat
where gcc
where g++
where gdb
```

`where`는 찾은 순서대로 보여주며, **맨 위 줄이 실제로 실행되는 것**이다.

PowerShell에서는 `where`가 다른 명령(`Where-Object`)의 별칭이라 **아무것도 출력되지 않는다.** PowerShell에서는 `where.exe g++`로 실행한다.

예시 (이 저장소를 처음 만든 PC의 정리 전 상태)

```text
C:\Program Files\mingw64\bin\gcc.exe    ← 별도 설치한 MinGW. 현재 이것이 실행됨
C:\msys64\ucrt64\bin\gcc.exe            ← MSYS2 UCRT64. 뒤로 밀려 있음
```

PATH에서 찾는 순서

- Windows는 **시스템 변수 → 사용자 변수** 순서로 찾는다.
- **같은 변수 안에서는 목록 위쪽에 있는 경로가 먼저** 쓰인다.
- 위 예시는 두 경로가 모두 시스템 변수에 있었고, `C:\Program Files\mingw64\bin`이 목록에서 더 위에 있었던 경우다.

### 3-2. 기존 MinGW를 PATH에서 빼기

1. 시작 메뉴에서 **"시스템 환경 변수 편집"** → 환경 변수 (관리자 권한 필요)
2. **시스템 변수**의 `Path`에서 기존 MinGW 경로(예: `C:\Program Files\mingw64\bin`)를 삭제
3. **사용자 변수**의 `Path`에도 있으면 삭제
4. `C:\msys64\ucrt64\bin`이 시스템 변수나 사용자 변수 중 **한 곳에** 있는지 확인. 어디에도 없으면 2-4대로 추가
5. 같은 목록에서 `C:\msys64\ucrt64\bin`보다 위에 다른 컴파일러 경로가 없는지 확인
6. 명령 프롬프트와 VS Code를 모두 닫았다가 다시 연다

Qt에 딸린 MinGW(`C:\Qt\Tools\mingw...\bin`)가 PATH에 있다면 그것도 빼는 것을 권장한다. Qt Creator는 PATH와 상관없이 자기 컴파일러를 찾아 쓰므로 Qt 작업에는 영향이 없다.

### 3-3. 기존 MinGW 폴더 정리 (나중에)

- PATH에서 뺀 뒤 며칠 써보고 문제가 없으면 지운다.
- 설치 프로그램으로 설치했다면 "앱 및 기능"에서 제거, 압축을 풀어 넣었다면 폴더 삭제.
- 지우기 전에 예전 프로젝트의 빌드 스크립트나 IDE 설정이 그 경로를 직접 쓰고 있지 않은지 확인한다.

### 3-4. 기존 설정과 실행 파일 정리

- 예전에 만든 `.vscode` 설정 파일에 기존 MinGW 경로가 적혀 있으면 지우거나 8장대로 다시 만든다.
- 예전에 빌드한 `.exe`는 다시 빌드한다. 예전 런타임 DLL을 찾다가 실행되지 않을 수 있다.

---

## 4. 설치 확인

명령 프롬프트(cmd)를 **새로 열고** 실행한다.

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

## 5. Git 설치와 첫 설정

- **왜**: 과제 저장소를 관리한다. VS Code의 소스 제어 화면도 이 Git을 사용한다. 저장소를 만들거나 clone하려면 VS Code 설정보다 먼저 필요하다.
- 이미 설치되어 있는지 먼저 확인한다.

    ```bash
    git --version
    ```

    `git version 2.xx.x.windows.1`처럼 `windows`가 붙어 있으면 Git for Windows다.

- 없으면 [git-scm.com](https://git-scm.com)에서 **Git for Windows**를 설치한다. 설치 옵션은 기본값으로 둔다. Git Bash 터미널도 함께 설치된다.
    - MSYS2에서 `pacman -S git`으로도 설치할 수 있지만, 그 git은 MSYS2 터미널 안에서만 동작하고 명령 프롬프트와 VS Code에서는 찾지 못한다. 그래서 Git for Windows를 권장한다.
- 사용자 정보 설정 (PC마다 한 번). 자세한 설명은 [WORKFLOW.md](WORKFLOW.md) 0-1.

    ```bash
    git config --global user.name "이름"
    git config --global user.email "GitHub 계정 이메일"
    ```

---

## 6. 저장소 준비

VS Code 설정(`.vscode`)은 저장소 폴더 안에 만들어지므로, 저장소 폴더를 먼저 준비한다.

### 6-1. 저장소를 처음 만드는 경우

[WORKFLOW.md](WORKFLOW.md)의 "0. 처음 한 번: 저장소 만들기"를 따른다.

### 6-2. 이미 있는 저장소를 다른 PC에서 쓰는 경우

```bash
git clone https://github.com/<계정>/cpp-study.git
```

- 현재 폴더 아래에 `cpp-study` 폴더가 생기고, 원격 저장소(`origin`)가 자동으로 연결된다.
- 경로에 공백이나 한글이 없는 폴더에서 실행하는 것을 권장한다.
- 저장소에 `.vscode/tasks.json`, `launch.json`, `.gitignore`가 이미 들어 있으므로 8장은 확인만, 9장은 하지 않아도 된다.

---

## 7. VS Code와 C/C++ 확장 설치

- **왜**: VS Code는 편집기이고, C++ 코드 분석과 디버깅 기능은 C/C++ 확장이 추가한다. 실제 컴파일은 2장에서 설치한 GCC가 한다.
1. [code.visualstudio.com](https://code.visualstudio.com)에서 설치
2. 확장(Extensions, `Ctrl+Shift+X`)에서 **C/C++** (게시자: Microsoft, ID `ms-vscode.cpptools`) 설치
3. **IntelliSense 컴파일러 지정**: `Ctrl+Shift+P` → `C/C++: Select IntelliSense Configuration` → `C:\msys64\ucrt64\bin\g++.exe` 선택
    - **왜**: 편집기의 자동완성과 빨간 줄 표시가 실제 빌드와 같은 컴파일러 기준으로 동작하게 한다. 빌드 자체에는 영향이 없다.

---

## 8. 빌드·디버그 설정 (`tasks.json`, `launch.json`)

`.vscode` 폴더의 두 파일이 VS Code의 빌드와 디버그 방법을 정한다. 두 파일 모두 VS Code로 **연 폴더(`cpp-study`)**에 적용된다.

| 파일 | 역할 |
| --- | --- |
| `tasks.json` | 실행할 작업(명령) 목록. 여기서는 빌드 작업으로 쓴다. `Ctrl+Shift+B`로 실행 |
| `launch.json` | 디버거를 어떻게 시작할지. 어떤 `.exe`를, 어떤 디버거로, 시작 전에 무슨 작업을 먼저 할지. `F5`로 실행 |
| `c_cpp_properties.json` | 편집기 코드 분석(IntelliSense) 설정. 7장 3번에서 생성될 수 있으며 저장소에 올리지 않는다 |

설정은 `cpp-study` 전체에 적용되지만, 실제로 빌드하고 디버깅하는 대상은 **지금 편집기에서 열려 있는 파일의 폴더**다. 두 파일 모두 `${fileDirname}`(열린 파일이 있는 폴더)을 쓰기 때문이다. 그래서 과제마다 설정을 새로 만들지 않아도 된다.

```text
F5 → launch.json → preLaunchTask "C++: build folder"
                       → tasks.json에서 label이 같은 작업 실행 (빌드)
                   → main.exe를 GDB로 실행 (디버깅)
```

**clone한 저장소라면** 두 파일이 이미 있다. 8-1의 5~6번, 8-2의 4번만 해서 동작을 확인한다.

**`${...}` 변수는 `$`, `{`, `}`까지 정확히 적어야 한다.** `$`가 빠지면 VS Code가 변수로 바꾸지 않고 글자 그대로 쓴다. 예를 들어 `"cwd": "{fileDirname}"`이면 존재하지 않는 폴더에서 실행하려다 `Starting build...`에서 멈춘다.

### 8-1. tasks.json 만들기

1. VS Code에서 **파일 → 폴더 열기 (File → Open Folder...)**로 `cpp-study` 폴더 **자체**를 연다. 그래야 `.vscode`가 저장소 루트에 만들어진다.
2. 과제 폴더(예: `sec05_copy-ctor/hw1_tracer/`)에 테스트용 `main.cpp`를 만든다. `cpp-study` 루트가 아니라 과제 폴더에 만드는 이유는, 실제 과제 코드가 들어갈 위치와 같은 조건에서 확인하기 위해서다.

    ```cpp
    #include <iostream>
    int main() { std::cout << "hello" << std::endl; return 0; }
    ```

3. `main.cpp`를 편집기에 연 상태에서 **터미널 → 기본 빌드 작업 구성 (Terminal → Configure Default Build Task...)** → **C/C++: g++.exe 활성 파일 빌드 (C/C++: g++.exe build active file)** 선택
    - 목록 두 번째 줄의 `compiler:` 경로가 `C:\msys64\ucrt64\bin\g++.exe`인 항목을 고른다.
    - 다른 경로(예: `C:\Program Files\mingw64\...`)가 보이면 3장의 PATH 정리가 적용되지 않은 것이다. VS Code를 완전히 닫았다가 다시 연다.
4. `.vscode/tasks.json`이 생성된다. 아래 기준으로 고친다.

| 항목 | 생성된 기본값 | 고칠 값 | 이유 |
| --- | --- | --- | --- |
| `label` | `C/C++: g++.exe build active file` | `C++: build folder` | 작업의 이름이자 식별자. 작업 목록에 표시되고, `launch.json`의 `preLaunchTask`가 이 이름으로 부른다. 원래 이름은 "열린 파일 하나 빌드"라는 뜻이라 실제 동작과 맞추기 위해 바꾼다 |
| `command` | `C:/msys64/ucrt64/bin/g++.exe` | `g++` | PATH에서 찾게 해서 PC마다 경로가 달라도 동작 |
| `args`의 `${file}` | 열린 파일 하나만 빌드 | `${fileDirname}\\*.cpp` | 과제 폴더의 `.cpp`(예: `main.cpp`, `CIntArray.cpp`)를 모두 빌드 |
| `args`의 출력 이름 | `${fileDirname}\\${fileBasenameNoExtension}.exe` | `${fileDirname}\\main.exe` | 실행 파일 이름을 고정해서 `launch.json`이 찾기 쉽게 |
| `args`에 추가 | 없음 | `-std=c++17`, `-Wall`, `-Wextra` (`-g` 다음) | C++ 표준 지정, 경고를 최대한 켜서 실수 발견 |
| `options`의 `cwd` | `C:/msys64/ucrt64/bin` | `${fileDirname}` | 빌드를 과제 폴더 기준으로 실행. 컴파일러 폴더 기준이면 오류 메시지의 파일 경로가 엉뚱하게 나올 수 있다 |

`detail`은 설명용이라 그대로 두거나 지워도 된다.

참고 예시 (직접 고친 뒤 비교용)

```jsonc
{
    "version": "2.0.0",
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C++: build folder",
            "command": "g++",
            "args": [
                "-fdiagnostics-color=always",
                "-g",                              // 디버그 정보 포함 (GDB에 필요)
                "-std=c++17",
                "-Wall",
                "-Wextra",
                "${fileDirname}\\*.cpp",           // 열린 파일이 있는 폴더의 모든 .cpp
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

5. 저장 후 `main.cpp`를 연 상태에서 `Ctrl+Shift+B` → `Build finished successfully.`가 나오고 과제 폴더에 `main.exe`가 생기면 성공.
6. 터미널에서 실행해본다.

    ```bash
    cd sec05_copy-ctor/hw1_tracer
    ./main.exe
    ```

**(선택) 복사 생략 끄기 빌드**: 5섹션처럼 생성자 호출 순서를 원리대로 보고 싶을 때 쓴다. 위 작업을 복사해 `tasks` 배열에 하나 더 넣고, `label`을 `C++: build folder (no elide)`로, `-std=c++17`을 `-std=c++14`와 `-fno-elide-constructors` 두 줄로 바꾼다. `group`은 `"build"`로 둔다(기본 작업은 하나만). 실행은 **터미널 → 빌드 작업 실행 (Terminal → Run Build Task...)** 목록에서 선택.

### 8-2. launch.json 만들기

1. `main.cpp`를 연 상태에서 **실행 → 구성 추가 (Run → Add Configuration...)** → **C++ (GDB/LLDB)** 선택
2. `.vscode/launch.json`이 생성되는데, 처음에는 `"configurations": []`처럼 **비어 있다.** 기본 틀을 넣는다.
    - `[`와 `]` 사이에 커서를 두고 `Ctrl+Space` → **C/C++: (gdb) 시작 (C/C++: (gdb) Launch)** 선택
    - `(gdb)`는 GDB 디버거를 쓴다는 뜻, `Launch`는 VS Code가 프로그램을 새로 실행하면서 디버깅한다는 뜻이다. `(gdb) Attach`(실행 중인 프로그램에 붙기), `(Windows) Launch`(Visual Studio 디버거)와 혼동하지 않는다.
    - `Select debugger` 목록이 다시 뜨면 `Esc`로 닫고 위 방법을 쓴다.
3. 틀에는 `"enter program name, ..."`, `"/path/to/gdb"` 같은 자리표시자가 들어 있다. 아래 기준으로 고친다.

| 항목 | 틀의 값 | 고칠 값 | 이유 |
| --- | --- | --- | --- |
| `name` | `(gdb) Launch` | `C++: debug folder` | 실행 화면에 보이는 이름 |
| `program` | `enter program name, ...` | `${fileDirname}\\main.exe` | 8-1에서 만든 실행 파일 |
| `miDebuggerPath` | `/path/to/gdb` | `C:\\msys64\\ucrt64\\bin\\gdb.exe` | 사용할 디버거. MSYS2를 기본 경로에 설치하면 어느 PC든 같다 |
| `preLaunchTask` | 없음 (새로 추가) | `C++: build folder` | 디버깅 전에 자동으로 빌드. 8-1의 `label`과 **글자 하나까지 같아야** 한다 |

`cwd`(`${fileDirname}`), `externalConsole`(`false`)은 틀에 이미 맞게 들어 있다. `setupCommands`의 `disassembly-flavor intel` 항목은 어셈블리를 인텔 표기법으로 보여주는 설정이라 그대로 둬도 된다.

`preLaunchTask`는 `setupCommands`의 닫는 `]` 뒤에 **쉼표를 붙이고** 다음 줄에 추가한다.

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
                },
                {
                    "description": "Set Disassembly Flavor to Intel",
                    "text": "-gdb-set disassembly-flavor intel",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C++: build folder"
        }
    ]
}
```

4. `main.cpp`의 `std::cout` 줄 번호 왼쪽을 클릭해 중단점(빨간 점)을 찍고 `F5` → 빌드 후 그 줄에서 멈추면 성공.

| 키 | 동작 |
| --- | --- |
| `F10` | 다음 줄로 (함수 안으로 들어가지 않음) |
| `F11` | 함수 안으로 들어가기. 생성자, 소멸자 호출 시점을 따라갈 때 유용 |
| `Shift+F11` | 지금 함수에서 빠져나오기 |
| `F5` | 다음 중단점까지 계속 |
| `Shift+F5` | 디버깅 중지 |

왼쪽 VARIABLES 패널에서 지역 변수 값을 볼 수 있다.

### 8-3. 확인이 끝나면

테스트용 `main.cpp`와 `main.exe`는 지운다. 커밋할 때 `git status`에 테스트 파일이 섞이지 않도록 주의한다.

---

## 9. `.gitignore` 설정: 두 설정 파일만 저장소에 올리기

> 이 저장소의 `.gitignore`에는 이미 반영되어 있다. 새 저장소를 만들 때나 원리를 확인할 때 참고한다.

- **왜**: `tasks.json`, `launch.json`을 저장소에 올리면 다른 PC에서 clone만 해도 같은 빌드·디버그 설정을 쓸 수 있다. 나머지 `.vscode` 파일(`settings.json`, `c_cpp_properties.json`)은 PC나 개인 취향에 따른 설정이라 올리지 않는다.

```gitignore
.vscode/*
!.vscode/tasks.json
!.vscode/launch.json
```

- `.vscode/*`: `.vscode` 안의 파일을 모두 제외
- `!`: 제외에서 다시 포함

`.vscode/`(폴더 자체 제외)로 쓰면 `!` 규칙이 동작하지 않는다. git은 제외된 폴더 안을 아예 들여다보지 않기 때문이다. 그래서 `.vscode/*`(폴더 안의 파일 제외)로 써야 한다.

커밋 시점

- **저장소의 첫 커밋 전**이라면 따로 커밋하지 않고 첫 커밋에 함께 포함한다.
- **저장소를 이미 만들었다면** 별도로 커밋한다.

    ```bash
    git add .
    git status            # .gitignore, .vscode/tasks.json, .vscode/launch.json만 보이면 정상
    git commit -m "[setup] VS Code 빌드·디버그 설정 추가"
    git push
    ```

---

## 10. 업데이트 방법

몇 달에 한 번, 또는 보안 업데이트 공지가 있을 때 하면 충분하다.

| 대상 | 방법 |
| --- | --- |
| MSYS2, GCC, GDB | **MSYS2 UCRT64** 터미널에서 `pacman -Syu`. 터미널이 닫히면 다시 열어 한 번 더 |
| Git for Windows | Git Bash를 **관리자 권한**으로 열고 `git update-git-for-windows`, 또는 cmd에서 `winget upgrade --id Git.Git` |
| VS Code, C/C++ 확장 | VS Code가 자동으로 업데이트한다. 알림이 뜨면 재시작 |

GCC 메이저 버전이 바뀐 뒤에는 기존 `.exe`를 다시 빌드한다.

---

## 11. 문제 해결

| 증상 | 원인 | 해결 |
| --- | --- | --- |
| `'g++'은(는) 내부 또는 외부 명령...이 아닙니다` | PATH에 `C:\msys64\ucrt64\bin`이 없거나, 창을 다시 열지 않음 | 2-4 확인 후 명령 프롬프트, VS Code 모두 다시 열기 |
| PowerShell에서 `where g++`가 아무것도 출력하지 않음 | PowerShell에서 `where`는 `Where-Object`의 별칭 | `where.exe g++`로 실행하거나 cmd에서 확인 |
| PATH를 고쳤는데 `where g++` 맨 위가 여전히 다른 경로 | 시스템 변수 PATH에 다른 MinGW가 남아 있거나, 목록에서 더 위에 있음 | 3-2대로 시스템 변수 확인 |
| `Ctrl+Shift+B` 후 `Starting build...`에서 멈춤 | `tasks.json`의 `${...}` 변수 오타 (예: `"{fileDirname}"`처럼 `$` 누락) | 멈춘 작업을 **터미널 → 작업 종료 (Terminal → Terminate Task...)**로 중단한 뒤, `$`, `{`, `}` 확인 |
| 같은 명령을 터미널에서 직접 실행하면 되는데 VS Code 빌드만 안 됨 | VS Code 작업 설정 문제 | `tasks.json` 오타 확인. 그래도 안 되면 `command`를 `C:/msys64/ucrt64/bin/g++.exe` 전체 경로로 |
| 빌드는 되는데 편집기에 빨간 줄이 뜸 | IntelliSense가 다른 컴파일러나 다른 C++ 표준 기준으로 분석 중 | 7장 3번 다시 실행, `C/C++: Edit Configurations (UI)`에서 C++ 표준을 `c++17`로 |
| `*.cpp: No such file or directory` | `.cpp`가 아닌 파일(예: `README.md`, `tasks.json`)을 연 상태로 빌드 | 빌드할 과제 폴더의 `.cpp` 파일을 연 상태에서 `Ctrl+Shift+B` |
| 다른 과제 폴더가 빌드됨 | `${fileDirname}`은 **지금 열려 있는 파일**의 폴더 | 빌드할 과제의 파일을 클릭한 뒤 빌드 |
| `F5` 시 `preLaunchTask ... 을(를) 찾을 수 없습니다` | `launch.json`의 `preLaunchTask`와 `tasks.json`의 `label`이 다름 | 두 값을 똑같이 맞추기 |
| `main.exe: Permission denied` 또는 빌드 시 파일 쓰기 실패 | 이전에 실행한 `main.exe`가 아직 실행 중 | 실행 중인 프로그램을 종료(`Ctrl+C`)하거나 디버깅을 중지한 뒤 다시 빌드 |
| `git add` 때 `LF will be replaced by CRLF` 경고 | 줄 끝 방식 차이(리눅스 LF, Windows CRLF)를 Git이 자동 변환한다는 안내 | 무시해도 된다. [WORKFLOW.md](WORKFLOW.md) 0-3 참고 |
| 출력의 한글이 깨짐 | 소스 파일은 UTF-8인데 콘솔은 CP949 | 실습 출력은 영문으로 쓰거나, 터미널에서 `chcp 65001` 실행 후 다시 실행 |
| `msys-2.0.dll이 없어` 실행되지 않음 | MSYS 환경 GCC(접두어 없는 `gcc` 패키지)로 빌드됨 | 1장 주의사항 확인, UCRT64 툴체인으로 다시 빌드 |

---

## 참고

- [Using GCC with MinGW – VS Code 공식 문서](https://code.visualstudio.com/docs/cpp/config-mingw)
- [MSYS2](https://www.msys2.org)
