# 시스템 보안 및 모니터링 (B1-1)
## 1단계: 인프라 기본 보안 및 네트워크 통제

```
요구사항 1-1: SSH 기본 포트(22)를 20022로 변경 []

요구사항 1-2: Root 계정의 원격 접속(PermitRootLogin)을 차단 []

요구사항 1-3: 방화벽(UFW 또는 firewalld)을 활성화하고, 인바운드 정책은 기본 차단(Deny)으로 설정 []

요구사항 1-4: 방화벽에서 관리용 포트(TCP 20022)와 서비스용 포트(TCP 15034)만 예외적으로 허용 []
```
### Docker 기반으로 우분투 실행 및 초기 설정
- `docker run -itd --privileged --name [컨테이너이름] [이미지이름] bash` B1_ubuntu 라는 이름으로 컨테이너 생성 및 실행
    - --privileged 옵션 통해 커널 기능 활성화
    - systemctl 기능 사용하기 위함

----


## 2단계: 역할 기반 계정 및 그룹 체계 구성 (IAM)
```
요구사항 2-1: 목적에 맞게 3개의 일반 사용자 계정(agent-admin, agent-dev, agent-test)을 생성할 것.

요구사항 2-2: 권한 분리를 위해 2개의 그룹(agent-common, agent-core)을 생성할 것.

요구사항 2-3: 각 계정의 역할에 맞게 그룹을 할당할 것 (예: agent-admin과 dev는 core 그룹 포함, test는 제외 등).
```
## 3단계: 디렉토리 구조 생성 및 최소 권한(ACL) 부여 (File System)

```
요구사항 3-1: AGENT_HOME 경로 아래에 목적별 디렉토리(upload_files, api_keys, bin) 및 로그 디렉토리(/var/log/agent-app)를 생성할 것.

요구사항 3-2: 공용 디렉토리(upload_files)는 agent-common 그룹에 R/W 권한을 부여할 것.

요구사항 3-3: 보안 디렉토리(api_keys) 및 로그 디렉토리는 핵심 인력인 agent-core 그룹에만 R/W 권한(770)을 부여하여 타 사용자의 접근을 원천 차단할 것.
```

## 4단계: 애플리케이션 환경 및 보안 키 구성 (App Environment)

```
요구사항 4-1: 지정된 경로에 인증용 API 키 파일(t_secret.key)을 생성하고 텍스트를 입력할 것.

요구사항 4-2: 키 파일의 권한을 640으로 설정하여 운영/개발자만 읽을 수 있도록 보호할 것.

요구사항 4-3: 애플리케이션 구동에 필요한 5가지 환경 변수(AGENT_HOME, AGENT_PORT, AGENT_UPLOAD_DIR, AGENT_KEY_PATH, AGENT_LOG_DIR)를 운영자(agent-admin) 계정에 설정할 것.
```

## 5단계: 서비스 구동 및 정상 동작 검증 (Service Execution)

```
요구사항 5-1: 반드시 일반 계정(agent-admin)으로 Python 애플리케이션을 실행할 것 (Root 실행 금지).

요구사항 5-2: 앱 실행 시 출력되는 'Boot Sequence 5단계'가 모두 [OK]로 통과되는지 확인할 것.

요구사항 5-3: 최종적으로 Agent READY 메시지가 출력되며, 15034 포트가 LISTEN 상태가 되는지 점검할 것.
```


## 6단계: 시스템 관제 스크립트 구현 (Monitoring Automation)

```
요구사항 6-1: Bash 쉘 스크립트(monitor.sh)를 작성하여 앱 프로세스 생존 여부 및 포트(15034) 상태를 체크할 것 (실패 시 exit 1 처리).

요구사항 6-2: CPU, 메모리, 디스크 사용률을 실시간으로 수집하고, 방화벽 활성화 상태를 점검할 것.

요구사항 6-3: 자원 사용량이 임계값(CPU 20%, MEM 10%, DISK 80%)을 초과하거나 방화벽이 꺼져있을 경우 [WARNING] 문구를 출력하도록 구현할 것.

요구사항 6-4: 스크립트 작성은 개발자(agent-dev)가, 실행 권한(750)은 그룹(agent-core)에 맞게 설정하여 무단 수정을 방지할 것.
```

## 7단계: 주기적 모니터링 적용 및 로그 라이프사이클 관리 (Scheduling & Rotation)

```
요구사항 7-1: 운영자(agent-admin)의 crontab을 활용하여 작성한 관제 스크립트가 매 1분마다 자동 실행되도록 등록할 것.

요구사항 7-2: 모니터링 결과가 지정된 포맷([시간] PID CPU MEM DISK)에 맞춰 monitor.log 파일에 지속적으로 누적되는지 검증할 것.

요구사항 7-3: 디스크 풀(Disk Full) 장애를 방지하기 위해, 로그 파일이 일정 용량(10MB)을 초과하면 백업 분리하고 최대 10개까지만 보관하는 Log Rotation 로직을 적용할 것.
```