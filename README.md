# Codyssey Week1 입학연수 과제

Docker와 Git/GitHub로 개발 환경을 만들고, 터미널·권한·컨테이너·포트·마운트·볼륨을 직접 실습한 기록입니다.

## 1. 실행 환경

| 항목 | 내용 |
| --- | --- |
| OS | Windows + WSL2 (Ubuntu 26.04 LTS) |
| Docker | Docker Desktop 4.96.3 (Docker 29.9.2, WSL2 백엔드) |
| Git | 2.53.0 |
| 편집기 | VSCode |
| 작업 폴더 | ~/codyssey/week1 |

![환경](docs/screenshots/01-environment.png)

## 2. 수행 체크리스트

- [x] 터미널 기본 조작
- [x] 파일/폴더 권한 변경
- [x] Docker 설치 및 점검
- [x] 컨테이너 실행/관리
- [x] Dockerfile 작성 및 빌드
- [x] 포트 매핑
- [x] 바인드 마운트
- [x] 볼륨 영속성
- [x] Git/GitHub 연동
- [x] README 작성

## 3. 수행 로그와 검증

### 3-1. 터미널 기본 조작
`pwd`, `mkdir`, `cd`, `touch`, `echo`, `cat`, `cp`, `mv`, `rm`을 `~/codyssey/week1/practice`에서 실습했습니다.

![터미널1](docs/screenshots/02-terminal-basic-1.png)
![터미널2](docs/screenshots/02-terminal-basic-2.png)

### 3-2. 권한
`ls -l`로 확인한 뒤 `chmod 600 a.txt`, `chmod 700 mydir`로 변경했습니다.

![권한](docs/screenshots/03-permission.png)

### 3-3. Docker 점검
`docker images`, `docker ps`, `docker ps -a`로 상태를 확인했습니다.

![점검](docs/screenshots/04-docker-check.png)

### 3-4. 컨테이너 실행/관리
- `docker run hello-world`
- `docker run -it ubuntu bash`
- `docker run -d --name u1 ubuntu sleep infinity` 후 `docker exec -it u1 bash`
- `docker logs`, `docker stats --no-stream`, `docker stop`, `docker rm`

핵심 확인: `docker exec`로 들어갔다 `exit`해도 컨테이너는 계속 Up 상태입니다.

![hello](docs/screenshots/05-hello-world.png)
![ubuntu](docs/screenshots/05-ubuntu-inside.png)
![exec vs run](docs/screenshots/05-exec-vs-run.png)
![exec vs run 2](docs/screenshots/05-exec-vs-run-2.png)
![logs stats](docs/screenshots/05-logs-stats.png)
![logs stats 2](docs/screenshots/05-logs-stats-2.png)

### 3-5. Dockerfile
```
FROM nginx:alpine
LABEL org.opencontainers.image.title="my-custom-nginx"
ENV APP_ENV=dev
COPY site/ /usr/share/nginx/html/
```
`docker build -t my-web:1.0 .` 후 `docker run --rm my-web:1.0 printenv APP_ENV`로 `dev`를 확인했습니다.

![Dockerfile](docs/screenshots/06-dockerfile.png)
![build](docs/screenshots/06-build.png)

### 3-6. 포트 매핑
`-p 8080:80`, `-p 8081:80`로 같은 이미지를 두 포트에 띄우고 `curl`과 브라우저로 확인했습니다.

![8080](docs/screenshots/07-port-8080.png)
![8081](docs/screenshots/07-port-8081.png)
![터미널](docs/screenshots/07-port-terminal.png)

### 3-7. 바인드 마운트
`-v "$(pwd)/site":/usr/share/nginx/html`로 연결한 뒤, 컨테이너를 다시 만들지 않고 호스트의 `index.html`만 수정했더니 화면이 바뀌었습니다.

![before](docs/screenshots/08-bind-before.png)
![after](docs/screenshots/08-bind-after.png)

### 3-8. 볼륨
`docker volume create mydata`로 만든 볼륨에 `hello.txt`를 저장하고, 컨테이너(vol1)를 삭제한 뒤 새 컨테이너(vol2)에 같은 볼륨을 연결하자 데이터가 남아 있었습니다.

![before](docs/screenshots/09-volume-before.png)
![delete](docs/screenshots/09-volume-delete.png)
![after](docs/screenshots/09-volume-after.png)

### 3-9. Git / GitHub
`git config`, `git init`, `git add .`, `git commit`, `git remote add origin`, `git push -u origin main` 순서로 진행했습니다.

![config](docs/screenshots/10-git-config.png)
![repo](docs/screenshots/11-github-repo.png)
![push](docs/screenshots/13-git-push.png)

## 4. 트러블슈팅

### 4-1. `ls: command not found`
- 증상: `ls`를 입력하면 command not found
- 원인: Ubuntu 26.04의 Rust 기반 coreutils에서 `/usr/bin/ls` 링크가 없었음
- 확인: PATH는 정상, `/usr/lib/cargo/bin/coreutils/ls`는 실행됨
- 해결: `alias ls="/usr/lib/cargo/bin/coreutils/ls"`를 `~/.bashrc`에 추가

![ls](docs/screenshots/02-troubleshoot-ls.png)

### 4-2. `docker build` 실패
- 증상: build 명령이 오류로 끝남
- 원인: `my-web:1.0.`처럼 빌드 컨텍스트 점(`.`) 앞에 띄어쓰기가 없었음
- 해결: `docker build -t my-web:1.0 .`로 수정

### 4-3. `pull access denied`
- 증상: `docker run`에서 pull access denied
- 원인: 이미지 이름이 `my web:1.0`으로 띄어쓰기가 들어가 Docker Hub에서 찾으려 함
- 해결: `my-web:1.0`으로 수정

### 4-4. `container is not running`
- 증상: `docker exec`에서 container is not running
- 원인: `sleep infinty` 오타로 컨테이너가 바로 종료(Exited 1)
- 확인: `docker ps -a`의 COMMAND 칸
- 해결: `docker rm -f vol2` 후 `sleep infinity`로 재생성

### 4-5. `invalid mode: data`
- 증상: `docker run`에서 invalid mode: data
- 원인: `-v mydata:/data:data`처럼 콜론이 두 번 들어가 `data`가 옵션으로 해석됨
- 해결: `-v mydata:/data`로 수정

### 4-6. `Could not resolve host: github.com`
- 증상: `git push` 실패, `ping 8.8.8.8`도 100% packet loss
- 원인: WSL 네트워크 일시 단절
- 해결: PowerShell에서 `wsl --shutdown` 후 재시작, `ping`으로 복구 확인

### 4-7. `Authentication failed`
- 증상: Password authentication is not supported for Git operations
- 원인: GitHub은 비밀번호 대신 토큰(PAT)만 허용
- 해결: Personal access token(repo 권한) 발급 후 Password 칸에 입력

## 5. 학습 정리 (내 말로 직접 채우기)

- 절대경로와 상대경로의 차이:
- 권한 755와 644의 의미:
- 이미지와 컨테이너의 차이:
- 포트 매핑이 필요한 이유:
- 볼륨이 필요한 이유 (바인드 마운트와의 차이):
- Git과 GitHub의 차이:

## 6. 재현 방법

```
cd my-web
docker build -t my-web:1.0 .
docker run -d -p 8080:80 --name my-web-8080 my-web:1.0
curl http://localhost:8080
```

## 7. 정리 명령

```
docker rm -f my-web-8080 my-web-8081
docker volume rm mydata
```
