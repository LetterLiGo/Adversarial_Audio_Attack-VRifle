# VRifle

This is the implementation of "Inaudible Adversarial Perturbation: Manipulating the Recognition of User Speech in Real Time"

We thank the author of [deepspeech2-pytorch-adversarial-attack](https://github.com/Kaminyou/deepspeech2-pytorch-adversarial-attack) for providing a good basis for our code, which attacks deepspeech2 model.

We also thank the contributors of [deepspeech.pytorch](https://github.com/SeanNaren/deepspeech.pytorch) for establishing a easy-to-use DeepSpeech Framework.

<!-- The pytorch version STFT algorithm is from [this repo](https://github.com/pseeth/torch-stft). -->
## Citation
If you think this repo helps you, please consider cite in the following format.
```latex
@inproceedings{li2024vrifle,
  title={Inaudible Adversarial Perturbation: Manipulating the Recognition of User Speech in Real Time},
  author={Li, Xinfeng and Yan, Chen and Lu, Xuancun and Zeng, Zihan and Ji, Xiaoyu and Xu, Wenyuan},
  booktitle={NDSS},
  year={2024}
}
```

## Get Start
Several dependencies required to be installed first. Please follow the instruction in [DeepSpeech 2 PyTorch](https://github.com/SeanNaren/deepspeech.pytorch) to build up the environments.</br>
It is recommended to setup your folders of [DeepSpeech 2 PyTorch](https://github.com/SeanNaren/deepspeech.pytorch) in the following structure.
```
ROOT_FOLDER/
├── this_repo/
│   ├──main_vrifle.py
│   └──...
├──deepspeech.pytorch/
│   ├──models/
│   │   └──librispeech/
│   │       └──librispeech_pretrained_v2.pth
│   └──...

```
Then, you should download the DeepSpeech pretrained model from this [link](https://github.com/SeanNaren/deepspeech.pytorch/releases) provided by the [DeepSpeech 2 PyTorch](https://github.com/SeanNaren/deepspeech.pytorch)

## Introduction
Deep Speech 2 <sup>[1]</sup> is a modern ASR system, which enables end-to-end training as spectrogram is directly utilized to generate predicted sentence. In this work, PGD (Projected gradient descent) and FGSM (Fast Gradient Sign Method) algorithms are implemented to conduct adversarial attack against this ASR system.

[1] Amodei, D., Ananthanarayanan, S., Anubhai, R., Bai, J., Battenberg, E., Case, C., ... & Zhu, Z. (2016, June). Deep speech 2: End-to-end speech recognition in english and mandarin. In International conference on machine learning (pp. 173-182).

## Preparation
1. Download the [Fluent Speech Command Dataset](https://www.kaggle.com/code/kerneler/starter-fluent-speech-corpus-76b2fe6d-6/data)
2. If you want to speed up the optimization on 3090 GPU. Turn to [Support DeepSpeech on 3090 GPUs (NVIDIA)](#jump)

## Usage
It is easy to perturb the original raw wave file to generate desired sentence with `main_vrifle.py`.
```script
python main_vrifle.py --attack_type Mute_robust --device 0

python main_vrifle.py --attack_type Universal_robust --device 0
```
Actually, several parameters are available to make your adversarial attack better. You may tune hypyerparameters such as `epsilon`, `alpha`, and `PGD_iter` to adjusted for better results. For the details, please refer to `main_vrifle.py` and `vrifle_attack.py`.

## <span id="jump"> Support DeepSpeech on 3090 GPUs (NVIDIA)</span>
**Through our many attempts and googles, the following setup details are formed :)**

### Install Deepspeech.pytorch
1. Download [deepspeech.pytorch](https://github.com/SeanNaren/deepspeech.pytorch/archive/refs/tags/V2.1.zip)
2. cd into the folder and then `pip install -r requirements.txt`
3. `pip install -e . # Dev install`
4. `pip install adversarial-robustness-toolbox[pytorch]`
5. `pip install torchaudio`
6. `git clone https://github.com/SeanNaren/warp-ctc.git`
7. You should replace the `#include <THC/THC.h>extern THCState* state`, which refers to https://blog.csdn.net/weixin_41868417/article/details/123819183修改binding.cpp`

**6. Install Warp-CTC**

- edit the CMakeLists.txt
```bash
# Before replacement
set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -gencode arch=compute_30,code=sm_30 -O2")
set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -gencode arch=compute_35,code=sm_35")

set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -gencode arch=compute_50,code=sm_50")
set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -gencode arch=compute_52,code=sm_52")

# After
set(CUDA_NVCC_FLAGS "${CUDA_NVCC_FLAGS} -gencode arch=compute_86,code=sm_86")
```

- Compilation
```bash
cd warp-ctc
mkdir build
cd build
cmake ..
make
cd ../pytorch_binding
```

- Modifying binding.cpp
```bash
## replace

#include <THC/THC.h>
extern THCState* state; 
void* gpu_workspace = THCudaMalloc(state, gpu_size_bytes);

## into
void* gpu_workspace = c10::cuda::CUDACachingAllocator::raw_alloc(gpu_size_bytes);


## replace
THCudaFree(state, (void *) gpu_workspace);
## into
c10::cuda::CUDACachingAllocator::raw_delete((void *) gpu_workspace);
```
- the last step
```bash
python setup.py install
```
- You should notice that the `--recursive` is required for a workable CTCdecode dependency
```bash
git clone --recursive git@github.com:parlance/ctcdecode.git
```