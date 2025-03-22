# 🔧 Vitis AI 튜토리얼 - PyTorch ResNet18 `.xmodel` 생성하기

## 📁 필요한 파일 목록
- `vitis-ai-master`, `archive.zip`, `run_all.sh` (GitHub 튜토리얼 사이트)
- `vitis-ai-3.5` 릴리즈
- `resnet18-f37072fd.pth` (Naver 등에서 다운로드)
- `quantize.py`

> ⚠️ `run_all.sh`는 `scripts`, `target` 등과 같은 상위 디렉토리에서 실행해야 하며, **WSL보다는 윈도우 파일 탐색기에서 압축 해제 권장**

---

## 🐳 Docker & WSL2 환경 설정

1. **WSL2 설치**
   ```bash
   wsl --list --verbose
   ```

2. **Docker 설치**  
   [공식 링크](https://www.docker.com/products/docker-desktop) 참고  
   설치 후 데몬 실행 필수

3. **Docker 관련 명령어**
   ```bash
   sudo apt update && sudo apt install unzip -y
   unzip '파일명' -d '디렉토리명'

   sudo apt install apt-transport-https ca-certificates curl software-properties-common
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | \
   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

   sudo apt update && sudo apt install docker-ce docker-ce-cli containerd.io -y
   docker --version
   ```

---

## 📦 Docker 이미지 다운로드 및 실행

```bash
docker pull xilinx/vitis-ai-cpu:latest  # CPU용
docker pull xilinx/vitis-ai-gpu:latest  # GPU용
```

> Vitis-AI docker 실행 예시:
```bash
docker run --rm -it \
  --shm-size=1g \
  -v ~/Vitis-AI-Tutorials/Tutorials/PyTorch-ResNet18/files:/workspace \
  xilinx/vitis-ai-cpu:latest
```

---

## 🧪 PyTorch 환경에서 학습 수행

```bash
conda activate vitis-ai-pytorch

cd ~/Vitis-AI
git clone --recurse-submodules https://github.com/Xilinx/Vitis-AI.git
cd Vitis-AI/docker
./docker_build.sh -t cpu -f pytorch
./docker_run.sh xilinx/vitis-ai-cpu
```

```bash
chmod +x train.py
python code/train.py \
 --batch-size 32 \
 --epochs 2 \
 --backbone resnet18 \
 --save-model \
 --data_root ./build/data/vcor \
 --save_dir ./build/float
```

> 학습 로그 예시:
```
Train Epoch: 1 [0/7267] Loss: 2.918763
...
Test set: Average loss: 0.8284, Accuracy: 1074/1550 (69.29%)
```

---

## 🎯 모델 양자화 (quantize.py)

```python
# quantize.py
import torch
from pytorch_nndct.apis import torch_quantizer
from torchvision import models

model = models.resnet18(pretrained=False)
model.load_state_dict(torch.load("resnet18-f37072fd.pth"))
model.eval()

dummy_input = torch.randn([1, 3, 224, 224])
quantizer = torch_quantizer('calib', model, (dummy_input,), device=torch.device("cpu"))
quant_model = quantizer.quant_model

for _ in range(100):
    quant_model(dummy_input)

quantizer.export_quant_config()
quantizer.export_xmodel(output_dir="quantize_result")
```

> 결과: `quantize_result` 폴더 내 `.xmodel` 생성 (양자화 완료 메시지 확인)

---

## ⚠️ 문제 발생 시 참고 사항
- `.xmodel`이 생성되지 않는 경우:
  - Docker 이미지 재설치 필요
  - `quantize.py` 내 device 설정 확인
  - `torch` 및 `pytorch_nndct` 버전 체크
