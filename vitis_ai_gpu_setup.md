# Vitis AI 3.5 GPU 환경 구성 및 오류 해결 기록

작성일: 2025-03-25  
작성자: rjsn@LAPTOP-FJ8QNV2R

---

## ✅ 1. 작업 디렉토리 설정

```bash
cd ~
mkdir -p Vitis-AI/tutorials/PyTorch-ResNet18
cd Vitis-AI
export WRK_DIR=$(pwd)
```

---

## ✅ 2. Dos-to-Unix 변환

스크립트 실행 시 오류 방지를 위해 dos2unix 변환 수행 (한 번만 필요):

```bash
sudo apt-get install dos2unix
cd ${WRK_DIR}/tutorials/PyTorch-ResNet18
for file in $(find . -name "*.sh"); do dos2unix ${file}; done
for file in $(find . -name "*.py"); do dos2unix ${file}; done
for file in $(find . -name "*.c*"); do dos2unix ${file}; done
for file in $(find . -name "*.h*"); do dos2unix ${file}; done
```

---

## ✅ 3. Docker 이미지 빌드

```bash
cd ${WRK_DIR}/docker
./docker_build.sh -t gpu -f pytorch
```

정상적으로 빌드되면 다음과 같은 이미지 확인 가능:

```bash
docker images
# 예시 출력
# xilinx/vitis-ai-pytorch-gpu   3.5.0.001-xxxxxx   xxx  ...   21.4GB
```

---

## ❌ 4. Docker 실행 시 오류 발생

### 오류 메시지

```
Error response from daemon: could not select device driver "" with capabilities: [[gpu]]
```

### 해결 방법

#### 1️⃣ NVIDIA Container Toolkit 미설치 확인

```bash
dpkg -l | grep nvidia-container
```

#### 2️⃣ NVIDIA Container Toolkit 설치

```bash
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)

# 키 및 소스 등록
curl -s -L https://nvidia.github.io/libnvidia-container/gpgkey |   sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

curl -s -L https://nvidia.github.io/libnvidia-container/$distribution/libnvidia-container.list |   sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#' |   sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list > /dev/null

# 패키지 설치
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
```

#### 3️⃣ Docker 재시작

```bash
sudo systemctl restart docker
```

---

## ✅ 5. GPU 연동 확인

```bash
docker run --rm --gpus all nvidia/cuda:12.2.0-devel-ubuntu22.04 nvidia-smi
```

성공 출력 예시:

```
NVIDIA-SMI ... CUDA Version: 12.2
...
GPU 0: NVIDIA GeForce RTX 2060
```

---

## ✅ 6. Docker 컨테이너 실행

```bash
cd ${WRK_DIR}
./docker_run.sh xilinx/vitis-ai-pytorch-gpu:3.5.0.001-1eed93cde
```

성공 시 아래와 같은 ASCII 아트 출력 확인:

```
==============================
██    ██ ██ ███████ ██ ███████ 
██    ██ ██ ██      ██ ██      
██    ██ ██ ███████ ██ ███████ 
...
Docker Image Version: 3.5.0.001-1eed93cde (GPU)
```

---

## ✅ 7. Vitis-AI 환경 진입 후 conda 활성화

```bash
conda activate vitis-ai-pytorch
cd /workspace/tutorials/PyTorch-ResNet18
```

---

## ✅ 8. 추가 패키지 설치 및 이미지 저장

### 패키지 설치

```bash
sudo su
conda activate vitis-ai-pytorch
pip install randaugment
pip install torchsummary
exit
```

### 커스텀 이미지 저장

```bash
sudo docker ps -l
# 예시: CONTAINER ID   IMAGE  ...
sudo docker commit -m "Add randaugment, torchsummary" <CONTAINER_ID> xilinx/vitis-ai-pytorch-gpu:3.5.0.001-custom
```

---

## 📝 요약

- ✅ Vitis AI 3.5 PyTorch GPU 도커 이미지 빌드 완료
- ✅ GPU 연동 성공 (RTX 2060)
- ✅ 추가 Python 패키지 설치 및 커스텀 이미지 저장 완료
- ❗ NVIDIA Container Toolkit 설치 필요 (처음 한 번만)
