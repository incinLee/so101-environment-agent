# Agentic 로봇 작업 환경 구축·검증 플랫폼

기백 팀 · 이주안 · 소속: 일반. SO-101의 가상 작업장을 만들고, 부품 이송 시연 수집·작은 AI 정책 학습·성공/실패 평가를 연결하는 **Sim2Real 개발 도우미의 가상 검증 단계**입니다.

## 소스 받기

전체 실행 소스는 [제출3_SO101_전체소스코드_공개본_20261006.zip](제출3_SO101_전체소스코드_공개본_20261006.zip)에 있습니다. ZIP을 **모두 압축 해제**하면 아래 두 폴더가 나란히 놓입니다. GitHub에 ZIP으로 게시하는 경우 개별 코드 열람은 압축을 해제한 뒤 가능합니다.

```text
so101-env-agent/   # backend 0.2.2: 환경 생성, MuJoCo, 분류, 시연, 학습, 평가
SO101_로컬앱/      # app 0.2.0: 목표/작업장 조건 입력과 실행 결과 화면
```

## Windows에서 실행

Python 3.12 설치 후, 압축을 푼 폴더에서 PowerShell을 엽니다. 첫 설치는 인터넷 연결이 필요합니다.

```powershell
Set-Location .\so101-env-agent
powershell -NoProfile -ExecutionPolicy Bypass -File .\scripts\run_demo.ps1
Set-Location ..\SO101_로컬앱
.\START_APP.cmd
```

프로그램 실행 중 CMD 창을 열어 두고 표시된 `http://127.0.0.1:포트/` 주소를 엽니다. 기본 포트는 8768입니다. 사용자 목표와 출발/도착 좌표를 각각 입력합니다. 문장의 숫자·방향을 좌표로 자동 변환하지 않습니다. 다른 폴더의 backend를 사용하려면 `start_app.ps1 -ProjectRoot "준비된 backend 폴더"`를 사용합니다.

설치 후 backend에서 8개 독립 CPU 작업장으로 시연을 수집하는 직접 명령:

```powershell
$env:PYTHONPATH = Join-Path (Get-Location) 'src'
.\.venv\Scripts\python.exe -m robot_env_agent agent --scene configs/scene.simulation.json --output-dir runs/my-parallel8 --epochs 60 --workers 8
```

매 실행마다 새로운 output-dir를 사용합니다. 관절각·부품 XYZ·목표 XYZ의 12차원 상태에서 6개 관절 목표각을 4스텝 단위로 예측하는 작은 Transformer를 CPU에서 모방학습합니다. 시연 총 9회, 학습/검증/시험 5/2/2회, 60epochs, 미니배치 64입니다. 8개 작업장은 시연 수집 프로세스의 수이며 강화학습이나 GPU 배치 실행이 아닙니다.

## 확인된 결과와 한계

- backend 시험 121건, 앱 시험 42건 통과. Windows CPU 검증입니다.
- scripted IK 기준 제어는 이송 **9/9 성공**, 학습 정책의 별도 시험은 **0/2 성공**입니다. 성공 영상은 학습 정책의 성과로 표시하지 않습니다.
- 동일 9개 seed의 시연 수집은 순차 10.000초, 8개 병렬 6.266초였습니다. 한 쌍의 CPU 수집 비교이며 전체 학습 시간이나 산업 성능을 뜻하지 않습니다.
- 목표 이해는 `tray_transfer`와 `response_calibration` 두 종류의 고정 작업 분류입니다. 범용 LLM, 임의 로봇 모델 복원, 임의 명령 실행은 미구현입니다.
- 실물 SO-101·Ubuntu·CUDA 실행, 실측 캘리브레이션 적용, 실제 Sim2Real 성공은 미검증입니다. 실물 전이를 위해 관절·좌표·단위 대응, 위치 관측, 제어 어댑터와 별도 현장 검증이 필요합니다.
- 부품/트레이/작업면 치수와 공개 모델의 물리값은 시뮬레이션 가정이며 사용자 장비 실측값이 아닙니다.

결과는 backend의 `runs/실행폴더/workflow_report.json`에 남습니다. 생성 장면은 `environment/scene.xml`, 시연별 판정·사진은 `baseline/seed-N/`, 정책 판정은 `policy_evaluation`에서 확인합니다. 기존 실패를 보존하며 학습 실패를 흐름 실행 성공으로 바꾸지 않습니다.

## 출처와 공개 범위

SO-101 MJCF/mesh는 [MuJoCo Menagerie 고정 커밋](https://github.com/google-deepmind/mujoco_menagerie/tree/3abca2e302625cec9429c49806121ea2855cef81/robotstudio_so101)에서 가져왔으며 Apache-2.0 원라이선스·README·CHANGELOG·파일별 SHA256을 보존했습니다. 자산 원본은 변경하지 않았습니다. Python/MuJoCo/NumPy/PyTorch 등 설치 의존성은 각각의 라이선스가 적용됩니다.

기존 자산은 공개 모델·라이브러리와 사용자가 제공한 보정 자료의 형식입니다. 이번 신규 개발은 환경 생성·검사, 두 목표 분류/도구 연결, 시연 수집, 작은 정책 학습/평가, 응답 추정, 로컬 화면과 기록 기능입니다. 실제 개발 증거는 2026-10-05~06이며 이전 날짜의 개발 이력을 만들지 않았습니다. Codex를 코드 작성·디버깅·문서 작성 보조로 사용했습니다. 과거 ACT 원본 코드·정책은 이번 코드에 통합하지 않았습니다.

사용자의 명시적 요청으로 공개 배포를 준비한 소스입니다. 원본 보정 ZIP·개인 자료·절대 개인 경로·인증정보·실행 로그·영상·학습 weights·가상환경은 제외했습니다. 자체 코드의 `Proprietary / All rights reserved` 표기는 유지하며, 저장소 공개가 수령자의 무제한 재배포·판매·재라이선스 허용을 뜻하지 않습니다. 제3자 자산의 Apache-2.0 권리는 별도로 보존됩니다.

저장소: https://github.com/incinLee/so101-environment-agent
