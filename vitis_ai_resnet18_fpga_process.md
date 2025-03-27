# Vitis AI 기반 PyTorch ResNet-18 모델의 FPGA(DPU) 배포 과정

본 문서는 **Ultra96-V2** FPGA 보드에 PyTorch ResNet-18 모델을 배포하는 전체 과정을 기록한 작업 로그입니다. 해당 작업은 AMD Xilinx의 Vitis AI 3.5 튜토리얼 중 `PyTorch-ResNet18` 예제를 기반으로 하며, 다음과 같은 단계로 진행되었습니다:

---

## ✅ 사용 환경 및 디렉토리 구조

- **작업 디렉토리**: `/workspace/tutorials/PyTorch-ResNet18/files`
- **작업 환경**: WSL2 + Ubuntu 20.04
- **GPU**: NVIDIA GeForce RTX 2060 (6GB)
- **사용 버전**
  - Vitis AI: 3.5
  - PyTorch: 1.13.1 + cu117
  - Python: 3.8.6

---

## ✅ 전체 흐름 요약

| 단계 | 설명 | 명령어/스크립트 |
|------|------|----------------|
| 1 | 데이터 준비 및 환경 구축 | run_all.sh |
| 2 | 모델 학습 | `run_train.sh` |
| 3 | 모델 테스트 | `run_test.sh` |
| 4 | 모델 양자화 | `run_quant.sh` |
| 5 | FPGA용 컴파일 (xmodel 생성) | `run_compile.sh` |
| 6 | Ultra96-V2 이식용 타르 파일 생성 | `target_zcu102.tar` |

---

## 1단계: 모델 학습

```bash
./scripts/run_train.sh
```

- 모델 구조: ResNet-18
- 데이터 경로: `./build/data/vcor`
- 저장 경로: `./build/float/color_last_resnet18.pt`
- 정확도: **86.516%**

출력 로그:
```
Test set: Average loss: 0.7967, Accuracy: 1341/1550 (86.516%)
```

---

## 2단계: 모델 테스트

```bash
./scripts/run_test.sh
```

> 초기 오류: `--test-batch-size` 인자 누락 → 수정 후 정상 작동

```bash
CUDA_VISIBLE_DEVICES=0 python code/test.py --batch-size 50 --test-batch-size 5 --backbone resnet18 ...
```

---

## 3단계: 모델 양자화 (Quantization)

```bash
./scripts/run_quant.sh
```

- 양자화 전 모델: `float/color_last_resnet18.pt`
- 양자화된 결과:
  - `quantized/ResNet_0_int.xmodel` ✅
  - `quantized/ResNet_int.pt`
  - `quantized/quant_info.json`

출력 로그:
```
Test set: Average loss: 0.8101, Accuracy: 1338/1550 (86.323%)
```

- GPU OOM 발생 시 CPU fallback 처리됨:
```bash
./scripts/run_quant.sh: line 24: Killed ...
```

---

## 4단계: 모델 컴파일 (Ultra96-V2 대상, ZCU102로 대체)

```bash
./scripts/run_compile.sh zcu102 ResNet_0_int.xmodel
```

정상 출력 로그:
```
[UNILOG][INFO] The compiled xmodel is saved to: ./build/compiled_zcu102/zcu102_ResNet_0_int.xmodel.xmodel
```

> `run_compile.sh`에서 `if [ $1 == zcu102 ]` → `if [ "$1" = "zcu102" ]`로 수정 필요 (구문 오류 해결)

---

## 5단계: Ultra96-V2 이식용 타르 파일 생성

```bash
cd build/
tar -cvf target_zcu102.tar compiled_zcu102/
```

파일:
- `target_zcu102.tar` 생성
- Ultra96-V2로 `scp` 또는 USB로 전송 가능

---

## 📸 주요 캡처/상황 정리

- 실시간 로그 에러 분석
- OOM 발생 시 fallback 처리 방식
- `.xmodel` 경로 존재 확인
- `run_compile.sh` 내 타겟 아키텍처 조건 분기 오류 수정

---

## 📌 비고

- Ultra96-V2는 `DPUCZDX8G` 아키텍처 기반이며 ZCU102와 동일한 컴파일 구조를 따름.
- 최종 `.xmodel`은 PYNQ 환경에서 사용 가능.
- 추후 `Jupyter Notebook` 또는 `DPU-PYNQ`에서 실행 필요.

---

(작성일: 2025년 3월 27일)
