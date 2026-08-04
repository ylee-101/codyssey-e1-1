# E1-1 | 개발자 작업 환경 구축

> 터미널, Docker, Git/GitHub를 이용해 누구나 같은 방식으로 실행·검증할 수 있는 개발 환경을 구성하는 실습입니다.

## 목표

- CLI로 파일과 디렉터리를 다루고, 권한의 의미를 확인한다.
- Docker 이미지와 컨테이너의 차이를 이해하고, 컨테이너의 상태를 관리한다.
- Dockerfile로 정적 웹 서버 이미지를 만들고 포트 매핑으로 접속을 확인한다.
- 바인드 마운트와 Docker 볼륨을 비교해 변경 반영과 데이터 영속성을 검증한다.
- Git으로 로컬 변경 이력을 관리하고 GitHub 원격 저장소와 연동한다.

## 실행 환경

| 항목 | 환경 |
| --- | --- |
| OS | macOS 26.0.1 |
| Shell | zsh (`/bin/zsh`) |
| Git | 2.50.1 (Apple Git-155) |
| Docker | OrbStack 또는 Docker Desktop에서 Docker Engine 실행 후 확인 |
| 원격 저장소 | `git@github.com:ylee-101/codyssey-e1-1.git` |

> macOS에서는 Docker 데몬을 직접 설치·실행하기보다 OrbStack 또는 Docker Desktop을 실행한 뒤 터미널에서 `docker` 명령을 사용한다. 버전은 실행 환경에 따라 다르므로 제출 전 아래 명령의 실제 출력으로 갱신한다.

```bash
docker --version
docker info
```

## 저장소 구조

```text
.
├── README.md
├── codyssey2-e1-1.pdf     # 과제 명세
└── src/                   # 수행 과정 스크린샷 및 웹 서버 소스 위치
```

## 수행 항목

### 완료 및 증거가 있는 항목

- [x] Git 저장소 생성 및 `origin` 원격 저장소 연결
- [x] Git 사용자 설정 확인
- [x] CLI로 `src` 디렉터리 생성, 파일 복사, 현재 위치와 목록 확인

### 제출 전 실제 실행 결과를 추가할 항목

- [ ] 파일 1개와 디렉터리 1개의 권한 변경 전·후 기록
- [ ] Docker 설치·데몬 점검 (`docker --version`, `docker info`)
- [ ] 이미지/컨테이너 운영 명령 (`docker images`, `docker ps -a`, `docker logs`, `docker stats`)
- [ ] `hello-world` 및 Ubuntu 컨테이너 실행
- [ ] Dockerfile로 웹 서버 이미지 빌드·실행
- [ ] 포트 매핑 접속 증거 (`curl` 또는 브라우저 주소창 포함 화면)
- [ ] 바인드 마운트 변경 반영 확인
- [ ] Docker 볼륨의 컨테이너 삭제 후 데이터 유지 확인

## 수행 로그

### 1. 터미널 기본 조작

과제 PDF와 스크린샷을 저장소에 정리하면서 디렉터리 생성, 복사, 이동, 현재 경로 및 목록 확인을 수행했다.

```bash
# 과제 파일을 저장소로 복사한 뒤 src 디렉터리에 보관
cp ~/Downloads/codyssey-e1-1.pdf ~/Desktop/github/
cd ~/Desktop/github
mkdir src
cp ./codyssey-e1-1.pdf ./src/

# 현재 작업 위치 및 파일 확인
cd ~/Desktop/github/codyssey-e1-1/src
pwd
ls
```

검증 결과: `pwd`로 현재 위치가 `.../codyssey-e1-1/src`임을 확인했고, `ls`에서 수집한 스크린샷 파일을 확인했다. [터미널 작업 증거](<src/스크린샷 2026-07-30 오후 4.34.36.png>)

### 2. Git 및 GitHub 연동

```bash
git config --list
git remote -v
git status
```

확인한 설정은 다음과 같다.

```text
remote.origin.url=git@github.com:ylee-101/codyssey-e1-1.git
branch.main.remote=origin
branch.main.merge=refs/heads/main
```

[Git 설정 증거](<src/스크린샷 2026-07-30 오후 1.03.56.png>)

## Docker 재현 절차

아래 명령은 과제 요구사항을 다시 검증할 때 사용하는 절차다. 이미지 이름, 컨테이너 이름, 포트는 다른 실행과 충돌하지 않도록 이 문서의 값으로 통일한다.

### 1. Docker 기본 동작 및 컨테이너 관리

```bash
docker run --rm hello-world
docker run -dit --name ubuntu-lab ubuntu:24.04 bash
docker exec ubuntu-lab bash -lc 'ls / && echo "container is running"'
docker ps
docker logs ubuntu-lab
docker stats --no-stream ubuntu-lab
docker stop ubuntu-lab
docker ps -a
docker rm ubuntu-lab
```

- `hello-world`의 성공 메시지로 이미지 다운로드와 컨테이너 실행을 확인한다.
- `ubuntu-lab`에서 `ls`, `echo`를 실행해 호스트와 분리된 컨테이너 내부에서 명령이 실행되는지 확인한다.
- `docker ps`는 실행 중인 컨테이너, `docker ps -a`는 종료된 컨테이너까지 보여 준다.

### 2. Dockerfile 기반 웹 서버

`src`에 정적 웹 페이지와 다음 Dockerfile을 둔다는 전제의 예시다. `nginx:alpine`을 베이스 이미지로 사용하고, 웹 콘텐츠만 교체한다.

```dockerfile
FROM nginx:alpine
LABEL org.opencontainers.image.title="codyssey-e1-1-web"
COPY src/ /usr/share/nginx/html/
EXPOSE 80
```

```bash
docker build -t codyssey-web:1.0 .
docker run -d --name codyssey-web -p 8080:80 codyssey-web:1.0
curl http://localhost:8080
docker logs codyssey-web
docker ps
```

`-p 8080:80`은 호스트의 8080 포트 요청을 컨테이너의 80 포트(NGINX)로 전달한다. 브라우저에서 `http://localhost:8080`에 접속하거나 `curl`의 HTML 응답을 기록해 포트 매핑을 검증한다.

### 3. 바인드 마운트: 호스트 변경 즉시 반영

```bash
docker run -d --name bind-web \
  -p 8081:80 \
  -v "$PWD/src:/usr/share/nginx/html:ro" \
  nginx:alpine

# 호스트의 src/index.html 수정 후
curl http://localhost:8081
```

`$PWD/src`는 호스트의 실제 디렉터리를 컨테이너에 연결한다. 따라서 호스트에서 `index.html`을 수정한 뒤 별도 이미지 빌드 없이 응답이 바뀌면 바인드 마운트가 동작한 것이다. `:ro`는 컨테이너가 호스트 파일을 수정하지 못하도록 읽기 전용으로 연결한다.

### 4. Docker 볼륨: 컨테이너 삭제 후에도 데이터 유지

```bash
docker volume create codyssey-data
docker run -d --name volume-before \
  -v codyssey-data:/data \
  ubuntu:24.04 sleep infinity
docker exec volume-before bash -lc 'echo "persistent data" > /data/result.txt && cat /data/result.txt'
docker rm -f volume-before

docker run -d --name volume-after \
  -v codyssey-data:/data \
  ubuntu:24.04 sleep infinity
docker exec volume-after cat /data/result.txt
```

두 번째 컨테이너에서도 `persistent data`가 출력되면 데이터가 컨테이너의 쓰기 계층이 아닌 Docker 볼륨에 저장되었음을 검증할 수 있다.

## 핵심 개념 정리

| 개념 | 정리 |
| --- | --- |
| 절대 경로 / 상대 경로 | 절대 경로는 `/Users/...`처럼 루트부터 시작하고, 상대 경로는 현재 위치를 기준으로 해석된다. `$PWD/src`는 실행 위치에 따라 달라지는 상대 대상의 절대 경로 표현이다. |
| 권한 | `r`, `w`, `x`는 각각 읽기·쓰기·실행 권한이다. 예를 들어 일반 파일 `644`는 소유자만 쓰기 가능하고, 디렉터리 `755`는 모든 사용자가 진입·목록 조회할 수 있다. |
| 이미지 / 컨테이너 | 이미지는 실행을 위한 불변 템플릿이고, 컨테이너는 그 이미지를 실행한 인스턴스다. 컨테이너를 삭제하면 컨테이너 내부 쓰기 데이터는 사라질 수 있다. |
| 포트 매핑 | 컨테이너의 80 포트는 호스트에서 자동 공개되지 않는다. `-p 호스트포트:컨테이너포트`로 호스트 요청을 전달한다. |
| 바인드 마운트 / 볼륨 | 바인드 마운트는 호스트 파일을 컨테이너에 직접 연결해 개발 중 변경 반영에 적합하다. 볼륨은 Docker가 관리하며 컨테이너를 삭제해도 데이터를 보존하는 데 적합하다. |
| Git / GitHub | Git은 로컬 변경 이력을 관리하는 분산 버전 관리 도구이고, GitHub는 원격 저장소를 통한 공유·협업 플랫폼이다. |

## 트러블슈팅

### `pwd src` 실행 오류

- 문제: `pwd src` 실행 시 `pwd: too many arguments`가 출력되었다.
- 원인 가설: `pwd`는 현재 작업 디렉터리를 출력하는 명령이며, 경로 인자를 받지 않는다.
- 확인: 인자 없이 `pwd`를 다시 실행해 현재 경로가 정상 출력되는 것을 확인했다.
- 해결: 경로 이동은 `cd src`, 현재 위치 확인은 `pwd`로 명령의 역할을 분리했다. [오류와 해결 증거](<src/스크린샷 2026-07-30 오후 4.34.36.png>)

### `docker: command not found`

- 문제: Docker Engine이 실행되지 않았거나 CLI가 설치되지 않은 환경에서는 `docker --version` 실행 자체가 실패한다.
- 원인 가설: macOS에는 Docker CLI/데몬이 기본 포함되지 않는다.
- 확인: OrbStack 또는 Docker Desktop의 실행 상태와 `docker info` 결과를 확인한다.
- 해결: OrbStack 또는 Docker Desktop을 설치·실행한 뒤 새 터미널에서 `docker --version`, `docker info`를 재실행한다. 이후 위 재현 절차의 실제 출력과 접속 화면을 이 문서에 추가한다.

## 보안 주의사항

- README, 터미널 로그, 스크린샷에 비밀번호·토큰·개인 키·인증 코드를 기록하지 않는다.
- `git config --list` 화면을 공유할 때 이메일 등 개인정보 노출 여부를 확인한다.
- 원격 저장소 주소는 공개 가능한 범위에서만 기록한다.
