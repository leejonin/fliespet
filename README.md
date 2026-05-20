# ConsoleApp1

C# 콘솔 애플리케이션으로, 파일을 여러 청크로 분할하여 병렬 전송하고(외부 저장소 시뮬레이션), 다시 청크를 불러와 원본 파일로 복원하는 예제입니다.

## 주요 기능

- 파일을 고정 크기 청크로 분할
- 워커 풀 기반 병렬 전송
- 전송 상태 및 진행률 집계
- Manifest 기반 청크 메타데이터 관리
- 청크 병렬 다운로드 후 파일 복원
- 복원 파일 체크섬 검증(SHA-256)

## 아키텍처

- `FileTransferController`
  - 업로드 오케스트레이션 담당
  - 청크 생성, 워커 실행, 완료 상태 반영
- `FileChunker`
  - 파일 분할 및 청크 체크섬 계산
- `TransferWorker`
  - 큐에서 청크를 가져와 병렬 업로드
  - 실패 시 재시도(백오프) 수행
- `InMemoryManifestStore`
  - 파일 단위 Manifest 저장/조회/갱신
- `LocalFolderStorageClient`
  - 외부 컴퓨터 저장소를 로컬 폴더로 시뮬레이션
- `RestoreService`
  - Manifest 기반 청크 병렬 다운로드 및 병합
  - 최종 파일 해시 검증

## 프로젝트 구조

```text
ConsoleApp1/
├─ Abstractions/
│  ├─ IChunker.cs
│  ├─ IManifestStore.cs
│  └─ IStorageClient.cs
├─ Models/
├─ Services/
├─ Storage/
└─ Program.cs
```

## 실행 방법

1. 프로젝트 디렉터리로 이동
2. 빌드 및 실행

```powershell
cd .\ConsoleApp1
dotnet run
```

## 실행 시 동작

- `bin/Debug/net10.0/input/sample.bin` 파일이 없으면 샘플 파일 자동 생성
- 파일을 청크로 나누어 `workerCount` 개 워커가 병렬 업로드
- 업로드 완료 후 `FileId` 기준으로 복원 수행
- 복원 파일을 `bin/Debug/net10.0/output/sample-restored.bin`에 생성

## 설정값(Program.cs)

- `chunkSizeInBytes`: 청크 크기 (기본 1MB)
- `workerCount`: 병렬 워커 수 (기본 4)
- 경로:
  - 입력: `input`
  - 원격 저장소(시뮬레이션): `remote-storage`
  - 복원 출력: `output`

## 참고 사항

- 현재 외부 전송은 `LocalFolderStorageClient`로 시뮬레이션되어 있습니다.
- 실제 외부 컴퓨터 전송으로 확장하려면 `IStorageClient` 구현을 HTTP/gRPC/SFTP 클라이언트로 교체하면 됩니다.
