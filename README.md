# PaddleOCR training for text recognition

## Chuẩn bị môi trường
```
pip install -q paddlepaddle-gpu "paddleocr>=2.0.1"
```

```python
import paddle
paddle.utils.run_check()
```
```
Running verify PaddlePaddle program ... 
PaddlePaddle works well on 1 GPU.
PaddlePaddle is installed successfully! Let's start deep learning with PaddlePaddle now.
```

```python
nvcc --version
nvidia-smi
```
```
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2023 NVIDIA Corporation
Built on Tue_Aug_15_22:02:13_PDT_2023
Cuda compilation tools, release 12.2, V12.2.140
Build cuda_12.2.r12.2/compiler.33191640_0
Thu Aug  8 05:30:38 2024       
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 535.104.05             Driver Version: 535.104.05   CUDA Version: 12.2     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  Tesla T4                       Off | 00000000:00:04.0 Off |                    0 |
| N/A   56C    P0              29W /  70W |    131MiB / 15360MiB |      0%      Default |
|                                         |                      |                  N/A |
+-----------------------------------------+----------------------+----------------------+
                                                                                         
+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
+---------------------------------------------------------------------------------------+
```
```python
git clone https://github.com/PaddlePaddle/PaddleOCR.git
```

## Folder Tree

```
📂 dataset
├── 📂 .ipynb_checkpoints
├── 📂 images
│   ├── 📄 000001.png
│   ├── 📄 000002.png
│   ├── 📄 000003.png
│   ├── 📄 000004.png
│   ├── 📄 000005.png
│   └── ...
├── 📂 original_images
│   ├── 📄 2aaa.png
│   ├── 📄 2aag.png
│   ├── 📄 2aam.png
│   ├── 📄 2aba.png
│   ├── 📄 2abe.png
│   └── ...
├── 📄 en_dict.txt
├── 📄 train_list.txt
├── 📄 v3_config.yml
├── 📄 v4_config.yml
└── 📄 val_list.txt
```

## Data Preprocessing

- ****Gán nhãn dữ liệu****
- ****Chia tập dữ liệu train và test****

## Cấu hình 

```python
Global:
  debug: false
  use_gpu: true
  epoch_num: 10
  log_smooth_window: 20
  print_batch_step: 10
  save_model_dir: ./output/rec_ppocr_v3
  save_epoch_step: 3
  eval_batch_step:
  - 0
  - 200
  cal_metric_during_train: true
  pretrained_model: /content/drive/MyDrive/Paddle_OCR/en_PP-OCRv3_rec_train/best_accuracy
  checkpoints: null
  save_inference_dir: ./output/save_inference
  use_visualdl: false
  infer_img: doc/imgs_words/ch/word_1.jpg
  character_dict_path: ./dataset/en_dict.txt
  max_text_length: 25
  infer_mode: false
  use_space_char: true
  distributed: true
  save_res_path: ./output/rec/predicts_v3_en.txt
Optimizer:
  name: Adam
  beta1: 0.9
  beta2: 0.999
  lr:
    name: Cosine
    learning_rate: 0.001
    warmup_epoch: 5
  regularizer:
    name: L2
    factor: 3.0e-05
Architecture:
  model_type: rec
  algorithm: SVTR_LCNet
  Transform: null
  Backbone:
    name: MobileNetV1Enhance
    scale: 0.5
    last_conv_stride:
    - 1
    - 2
    last_pool_type: avg
    last_pool_kernel_size:
    - 2
    - 2
  Head:
    name: MultiHead
    head_list:
    - CTCHead:
        Neck:
          name: svtr
          dims: 64
          depth: 2
          hidden_dims: 120
          use_guide: true
        Head:
          fc_decay: 1.0e-05
    - SARHead:
        enc_dim: 512
        max_text_length: 25
Loss:
  name: MultiLoss
  loss_config_list:
  - CTCLoss: null
  - SARLoss: null
PostProcess:
  name: CTCLabelDecode
Metric:
  name: RecMetric
  main_indicator: acc
  ignore_space: false
Train:
  dataset:
    name: SimpleDataSet
    data_dir: ./dataset/
    ext_op_transform_idx: 1
    label_file_list: ./dataset/train_list.txt
    transforms:
    - DecodeImage:
        img_mode: BGR
        channel_first: false
    - RecConAug:
        prob: 0.5
        ext_data_num: 2
        image_shape:
        - 48
        - 320
        - 3
        max_text_length: 25
    - RecAug: null
    - MultiLabelEncode: null
    - RecResizeImg:
        image_shape:
        - 3
        - 48
        - 320
    - KeepKeys:
        keep_keys:
        - image
        - label_ctc
        - label_sar
        - length
        - valid_ratio
  loader:
    shuffle: true
    batch_size_per_card: 20
    drop_last: true
    num_workers: 4
Eval:
  dataset:
    name: SimpleDataSet
    data_dir: ./dataset/
    label_file_list: ./dataset/val_list.txt
    transforms:
    - DecodeImage:
        img_mode: BGR
        channel_first: false
    - MultiLabelEncode: null
    - RecResizeImg:
        image_shape:
        - 3
        - 48
        - 320
    - KeepKeys:
        keep_keys:
        - image
        - label_ctc
        - label_sar
        - length
        - valid_ratio
  loader:
    shuffle: false
    drop_last: false
    batch_size_per_card: 10
    num_workers: 4
  batch_size_per_card: 10
```

## Training

```python
python '/content/drive/MyDrive/Paddle_OCR/PaddleOCR/tools/train.py' -c '/content/drive/MyDrive/Paddle_OCR/dataset/v3_config.yml'
```

## Inference

```python
ocr = PaddleOCR(use_angle_cls=True, lang='en', rec_model_dir=model_dir)

result = ocr.ocr(img_path, cls=False)
```

# 🌟 Reference:
- [PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR)
