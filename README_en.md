# AnyOCR

<a href="https://huggingface.co/anyforge/anyocr" target="_blank"><img src="https://img.shields.io/badge/%F0%9F%A4%97-HuggingFace-blue"></a>
<a href="https://www.modelscope.cn/models/anyforge/anyocr" target="_blank"><img alt="Static Badge" src="https://img.shields.io/badge/%E9%AD%94%E6%90%AD-ModelScope-blue"></a>
<a href=""><img src="https://img.shields.io/badge/Python->=3.6-aff.svg"></a>
<a href=""><img src="https://img.shields.io/badge/OS-Linux%2C%20Win%2C%20Mac-pink.svg"></a>
<a href=""><img alt="Static Badge" src="https://img.shields.io/badge/engine-cpu_gpu_onnxruntime-blue"></a>

```
    ___                ____  __________ 
   /   |  ____  __  __/ __ \/ ____/ __ \
  / /| | / __ \/ / / / / / / /   / /_/ /
 / ___ |/ / / / /_/ / /_/ / /___/ _, _/ 
/_/  |_/_/ /_/\__, /\____/\____/_/ |_|  
             /____/                     

```

English | [简体中文](./README.md)

## 1. Introduction

At present, we are very pleased to launch the Onnx format OCR tool 'AnyOCR' that is compatible with multiple platforms. Its core highlight is the use of ONNXRime as the inference engine, which ensures efficient and stable operation compared to PaddlePaddle inference engine.

- github地址：[AnyOCR](https://github.com/anyforge/anyocr)
- Hugging Face: [AnyOCR](https://huggingface.co/anyforge/anyocr)
- ModelScope: [AnyOCR](https://www.modelscope.cn/models/anyforge/anyocr)

## 2. Origin

The PaddlePaddle team has implemented an OCR tool based on PaddlePaddle in the PaddleOCR project, which has powerful performance and functionality. However, in certain scenarios, there are some issues with the speed and stability of the PaddlePaddle inference engine. So we collected a lot of new OCR data to fine tune and optimize PaddleOCR, and exported it in onnx format, directly using onnx runtime inference, avoiding the pitfalls of PaddlePaddle inference engine, and supporting CPU, GPU, etc.

PaddleOCR does not perform well on some new types of data or domain data, so we collected a lot of data for fine-tuning training, covering various fields, including:

- cc-ocr
- Industrial
- Medical treatment
- Physical examination
- Chinese
- English
- Paper
- Network
- Self built
- ETC.

Total dataset: greater than `385K`。

### Extended training

- train datasets：`385K`
- test datasets：`5k`
- ACC：`0.952`

### Model introduction

- Detection model: `anyocr_det_ch_v4_lite.onnx`, fine tuned and trained on our dataset by `ch_PP-OCRv4_det`.
- Recognition model: `anyocr_rec_v4_server.onnx`, fine tuned and trained on our dataset by `ch_PP-OCRv4_server_rec`.
- Direction classification: `anyocr_cls_v4.onnx`, sourced from `ch_ppocr_mobile_v2.0_cls` without training.
- Text character: `anyocr_keys_v4.txt`, derived from `ppocr/utils/ppocr_keys_v1.txt`.
- Larger and stronger: We have also trained a larger and stronger text recognition model that supports recognition of Chinese, English, and numbers. It supports recognition of over 15000 characters and some obscure characters, and can be applied for by email.


### evaluation

Self built evaluation datasets：`1.1K`

Extract 1150 pairs of untrained data for evaluation, covering Chinese, English, numbers, symbols, etc.

Our evaluation set and other OCR accuracy testing evaluations:

 - Our anyocr: 0.97
 - Paddleocr：0.92
 - Ali duguang ocr：0.86
 - GOT_OCR2.0：0.89
 - Olm-ocr: 0.46

## 3. Usage

### Install dependencies

```bash
## for cpu
pip install -r requirements.txt

## for gpu
pip install -r requirements-gpu.txt
```

### Method of use

```python
## simple
# use_det = True or False, if text detection
# use_cls = True or False, if text orientation cls
# use_rec = True or False, if text recognition

from anyocr.pipeline import anyocr

model = anyocr()

res = model.raw_completions('/to/your/image',use_cls=True,use_det=True)

print(res)


## return_word_box

from anyocr.pipeline import anyocr

model = anyocr()

res = model.raw_completions('/to/your/image',use_cls=True,use_det=True,return_word_box = True)



### custom model
from anyocr.pipeline import anyocr
from anyocr.pipeline import anyocrConfig


config = anyocrConfig(
    det_model_path = "anyocr/models/anyocr_det_ch_v4_lite.onnx",
    rec_model_path = "anyocr/models/anyocr_rec_v4_server.onnx",
    cls_model_path = "anyocr/models/anyocr_cls_v4.onnx",
    rec_keys_path = "anyocr/models/anyocr_keys_v4.txt"   
)
config = config.model_dump()
model = anyocr(config)

res = model.raw_completions('/to/your/image',use_cls=True,use_det=True)

print(res)
```

### Use paddleocr integration

```python
from paddleocr import PaddleOCR, draw_ocr

ocrmodel = PaddleOCR(
    use_gpu = False, # or True
    det_model_dir = "anyocr/paddlemodels/det/ch_PP-OCRv4_det_infer",
    cls_model_dir = "anyocr/paddlemodels/cls/ch_ppocr_mobile_v2.0_cls_infer",
    rec_model_dir = "anyocr/paddlemodels/rec/anyocr_rec_v4_server",
    rec_char_dict_path = "anyocr/paddlemodels/anyocr_keys_v4.txt",
    use_dilation = True,  
)
img_path = '/to/your/image'

result = ocrmodel.ocr(img_path, cls=True)
for idx in range(len(result)):
    res = result[idx]
    for line in res:
        print(line)

```

- If you have better text detection, text recognition can also use only a part of ours.
- You can also export the PaddleOCR model to onnx format and use AnyOCR inference, or you can fine tune the PaddleOCR model yourself and use AnyOCR inference.


### Configuration

```python
from pydantic import BaseModel

class anyocrConfig(BaseModel):
    text_score: float = 0.5   # Confidence level of text recognition results, range of values：[0, 1]
    use_det: bool = True  # if text detection
    use_cls: bool = True  # if text orientation cls
    use_rec: bool = True  # if text recognition
    print_verbose: bool = False # verbose
    min_height: int = 30  # The minimum height of the image (in pixels), below which the text detection stage will be skipped and subsequent recognition will be carried out directly.
    width_height_ratio: float = 8 # If the aspect ratio of the input image is greater than width_height_ratio, text detection will be skipped and subsequent recognition will be performed directly
    max_side_len: int = 2000 #  If the maximum edge of the input image is greater than max_side_len, the maximum edge will be reduced to max_side_len according to aspect ratio
    min_side_len: int = 30 # If the minimum edge of the input image is smaller than min_side_len, the minimum edge will be scaled to min_side_len according to aspect ratio
    return_word_box: bool = False # Whether to return the single character coordinates of the text
    
    det_use_cuda: bool = False  # if use gpu
    det_model_path: Optional[str] = None # text detection model path
    det_limit_side_len: float = 736 # Pixel values that limit the length of image edges
    det_limit_type: str = "min" # Limit the minimum or maximum edge length of the image to det_limit_side_len, with a value range of：[min, max]
    det_max_candidates:int = 1000 # Maximum number of candidate boxes
    det_thresh: float = 0.3  # The segmentation threshold for the text and background parts in the image. The larger the value, the smaller the text part will be. Value range：[0, 1]
    det_box_thresh: float = 0.5 # The threshold for whether the box obtained from text detection is retained, the larger the value, the lower the recall rate. Value range：[0, 1]
    det_unclip_ratio: float = 1.6 # Control the size of the text detection box, the larger the value, the larger the overall detection box. Value range：[1.6, 2.0]
    det_donot_use_dilation: bool = False # Do you want to use dilation? This parameter is used to perform morphological dilation on the detected text area
    det_score_mode: str = "slow"  # The method for calculating the score of a text box. The range of values is：[slow, fast]
    
    cls_use_cuda: bool = False  # if use gpu
    cls_model_path: Optional[str] = None #  text orientation cls model path
    cls_image_shape: List[int] = [3, 48, 192] # Image shape of input direction classification model (CHW)
    cls_label_list: List[str] = ["0", "180"] # The label for directional classification, 0 ° or 180 °, cannot be changed
    cls_batch_num: int = 6 # The batch size for batch inference is generally set to the default value. If it is too large, it may not significantly speed up the process and may result in poor performance. The default value is 6.
    cls_thresh: float = 0.9 # Confidence level of directional classification results. Value range：[0, 1]
    
    rec_use_cuda: bool = False  # if use gpu
    rec_keys_path: Optional[str] = None # text recognition cahr file path
    rec_model_path: Optional[str] = None # text recognition model path
    rec_img_shape: List[int] = [3, 48, 320] #  Image shape of input direction recognition model (CHW)
    rec_batch_num: int = 6 # The batch size for batch inference is generally set to the default value. If it is too large, it may not significantly speed up the process and may result in poor performance. The default value is 6.

```

## Buy me a coffee

- 微信(WeChat)

<div align="left">
    <img src="./zanshan.jpg" width="30%" height="30%">
</div>

## Special Thanks
- `paddleocr` Provide original models and fine-tuning tutorials
- Most of the source code comes from `RapidOCR`，I have made some personal changes

## Star History

[![Star History Chart](https://api.star-history.com/svg?repos=anyforge/anyocr&type=Date)](https://www.star-history.com/#anyforge/anyocr&Date)
