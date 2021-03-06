# ๐คธโโ๏ธImage Classification

โ์์ํ Daily Contributions๋ [์ด๊ณณ](https://www.notion.so/iloveslowfood/Stage-2-Image-Classification-58dbfca2e1ef4e36b8de6790b403ccba)์ ์๋ก๋๋์ด ์์ต๋๋ค:)



## Task Description

- ***Period.*** 2021.03.29~2021.04.08

- ***Problem Type.*** Classification - ๋ง์คํฌ/์ฑ๋ณ/์ฐ๋ น๋์ ๋ฐ๋ฅธ 18๊ฐ ํด๋์ค

- ***Metric.*** Macro F1 Score

- ***Data.*** ํ ๋ช๋น 7์ฅ(๋ง์คํฌ ์ฐฉ์ฉx1, ๋ฏธ์ฐฉ์ฉx1, ๋ถ์์  ์ฐฉ์ฉx5) ,์ด *2*,700๋ช์ ์ด๋ฏธ์ง. ํ ์ฌ๋๋น 384x512

  ![maskimg](https://github.com/iloveslowfood/ImageClassfication/blob/main/etc/maskimg.png?raw=true)



## Performances

### Scores

- Public LB.  F1 0.7706, Accuracy 81.3333%

- Private LB. F1 0.7604, Accuracy 81.0952%

  

### Best Model Configuration

#### I. Structure: *Ensemble VanillaEfficientNet with K-Fold CV*

K-Fold CV๋ฅผ ํตํด ํ์ต๋ VanillaEfficientNet ๋ชจ๋ธ์ ๊ฐ Fold๋ณ๋ก N๊ฐ์ฉ ์ ์ฅ, ์ถ๋ก  ๋จ๊ณ์์ ๊ฐ Fold๋ณ K๊ฐ์ ๋ชจ๋ธ์ ๋ถ๋ฌ์ TTA(Test Time Augmentation)๋ฅผ ์ ์ฉํ์ฌ ๋ชจ๋  ๊ฒฐ๊ณผ๊ฐ์ ์์๋ธํ์ต๋๋ค. Fold ์๋ฅผ 5๋ก, Fold๋ณ ๋ถ๋ฌ์ฌ ๋ชจ๋ธ ์๋ฅผ 2๋ก, TTA๋ฅผ 2๋ก ์ค์ ํ์ฌ ์ด 20(5x2x2)๊ฐ์ ์ถ๋ก  ๊ฒฐ๊ณผ๋ฅผ ์ฐ์ ํ๊ท ํ Soft Voting ์์๋ธ์ ์ฑ๋ฅ์ด ๊ฐ์ฅ ๋์์ต๋๋ค. (Private LB. F1 0.7604, Accuracy 81.0952%)

![ensemble_1](https://github.com/iloveslowfood/ImageClassfication/blob/main/etc/ensemble_1.png?raw=true)

![ensemble_2](https://github.com/iloveslowfood/ImageClassfication/blob/main/etc/ensemble_2.png?raw=true)

#### II. Hyper Parameters

์์๋ธ์ ํ์ฉํ VanillaEfficientNet ๋ชจ๋ธ์ ํ์ดํผ ํ๋ผ๋ฏธํฐ๋ ๋ค์๊ณผ ๊ฐ์ต๋๋ค.

```python
batch_size=32
epochs=๋ชจ๋ธ ๋ณ ์์ด
loss_type='labelsmoothingLoss'
lr=0.001
lr_scheduler='cosine' # cosine annealing warm restart
model_type='VanillaEfficientNet'
optim_type='adam'
seed=42
transform_type='tta'

# 'tta' transform
# train phase
transforms.Compose(
    [
        transforms.CenterCrop((384, 384)),
        transforms.RandomResizedCrop((224, 224)),
        RandAugment(2, 9), # (N, M): (# of transform candidates, # of changes)
        transforms.ToTensor(),
        transforms.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225]),
    ]
)

# test phase
transforms.Compose(
    [
        transforms.CenterCrop((384, 384)),
        transforms.RandomResizedCrop((224, 224)),
        transforms.ToTensor(),
        transforms.Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225]),
    ]
)
```



## Command Line Interface

### Train Phase

```python
>>> python train.py --task 'main' --model-type 'VanillaEfficientNet' --cv 5
```

18๊ฐ์ง ์นดํ๊ณ ๋ฆฌ๋ฅผ ๋ถ๋ฅํ๋ Main Task ์ด์ธ์ ๋ง์คํฌ ์ํ, ์ฐ๋ น๋(classification) ๋ฐ ์ฐ๋ น(regression), ์ฑ๋ณ์ 4๊ฐ์ง sub task์ ๋ํ ํ์ต์ ๋ชจ๋ ์ง์ํ๋ฉฐ, K-Fold CV ํ์ต ๋ํ ๊ฐ๋ฅํฉ๋๋ค. ์กฐ์  ๊ฐ๋ฅํ argument๋ ๋ค์๊ณผ ๊ฐ์ต๋๋ค.

- `task` : main task(*main*), ๋ง์คํฌ ์ํ(*mask*), ์ฐ๋ น๋(*ageg*), ์ฐ๋ น(*age*), ์ฑ๋ณ(*gender*)์ 5๊ฐ์ง task์ ๋ํ ํ์ต์ด ๊ฐ๋ฅํฉ๋๋ค. (default: *main*)

- `model_type` : ๋ถ๋ฌ์ฌ ๋ชจ๋ธ ์ํคํ์ณ๋ฅผ ์ ํํฉ๋๋ค. ์ง์ํ๋ ๋ชจ๋ธ ์ํคํ์ณ๋ ***VanillaEfficientNet***(`'VanillaEfficientNet'`), ***VanillaResNet***(`'VanillaResNet'`), ***MultiLabelTHANet*** (`'MultiLabelTHANet'`), ***MultiClassTHANet***(`'MultiClassTHANet_MK1'`), ***THANet***(`'THANet_MK1'`)์ด ์์ต๋๋ค. (default: `'VanillaEfficientNet'`)

- `load_state_dict`: ์ ์ฅ๋ ๋ชจ๋ธ์ ๋ถ๋ฌ์ ํ์ตํ  ๊ฒฝ์ฐ ์ ์ฅ๋ ํ์ผ์ ๊ฒฝ๋ก๋ฅผ ์๋ ฅํฉ๋๋ค. ์ ์ฅ๋ ํ๋ผ๋ฏธํฐ์ `model_type`์ด ์ผ์นํด์ผ ํฉ๋๋ค.

- `train/valid_root`: ํ์ต์ฉ ๋ฐ์ดํฐ์ ๊ฒ์ฆ์ฉ ๋ฐ์ดํฐ์ ๊ฒฝ๋ก๋ฅผ ์๋ ฅํฉ๋๋ค.

- `transform_type`: Augmentation์ ํ์ฉํ  Transform ์ข๋ฅ๋ฅผ ์๋ ฅํฉ๋๋ค. ***Base***(`'base'`), ***Random***(`'random'`), ***TTA***(`'tta'`), ***Face Crop***(`'facecrop'`)์ ์ง์ํฉ๋๋ค.

- `age_filter`: ์ผ์  ๋์ด ์ด์์ธ ์ธ๋ฌผ์ ์ฐ๋ น๋๋ฅผ 60๋ ์ด์ ์ฐ๋ น๋๋ก ๊ฐ์  ๋ณ๊ฒฝํฉ๋๋ค. 50๋ ํ๋ฐ์ ์ธ๋ฌผ๊ณผ 60๋ ์ธ๋ฌผ์ ์ฌ์ง์ ๋ถ๊ฐํ๊ธฐ๊ฐ ์ด๋ ค์ ์์ธก ์ฑ๋ฅ์ ์ง์ฅ์ ์ฃผ๋ ๊ฒฝ์ฐ๊ฐ ์์๊ธฐ ๋๋ฌธ์ ๊ณ ์ํ argument์๋๋ค. ๊ฐ๋ น,  `age_filter=58`๋ก ์ค์  ์ 58์ธ ์ด์์ธ ์ธ๋ฌผ ๋ชจ๋๊ฐ '60๋ ์ด์ ์ฐ๋ น๋' ๋ฒ์ฃผ๋ก ๊ฐ์  ๋ณ๊ฒฝ๋ฉ๋๋ค.

- `epochs`: ์ํญ์ ์ค์ ํฉ๋๋ค. (default: 30)

- `cv`: K-Fold CV๋ฅผ ํ์ฉํ  ๊ฒฝ์ฐ ์ฌ์ฉํ๋ argument๋ก, Fold ์๋ฅผ ์ค์ ํฉ๋๋ค. ์๋ ฅํ์ง ์๊ฑฐ๋ 1์ ์๋ ฅํ  ๊ฒฝ์ฐ ๋จ์ผ ํด๋๋ก ํ์ต, ์ฆ K-Fold CV๊ฐ ์งํ๋์ง ์์ต๋๋ค. (default: 1)

- `batch_size`: ๋ฐฐ์น ์ฌ์ด์ฆ๋ฅผ ์ค์ ํฉ๋๋ค. (default: 32)

- `optim_type`: ์ต์ ํ ํจ์๋ฅผ ์ค์ ํฉ๋๋ค. ***Adam***, ***AdamP***, ***SGD***, ***Momentum***์ ์ง์ํฉ๋๋ค. (default: `adam`)

- `loss_type`: ์์ค ํจ์๋ฅผ ์ค์ ํฉ๋๋ค. ***Label Smoothing Loss***(`'labelsmoothingloss'`), ***Focal Loss***(`'focalloss'`), ***Cross Entropy Loss***(`'crossentropyloss'`), ***MSE Loss***(`'mseloss'`), ***Smooth L1 Loss***(`'smoothl1loss'`)๋ฅผ ์ง์ํฉ๋๋ค. (default: `labelsmootingloss`)

- `lr`: Learning Rate๋ฅผ ์ค์ ํฉ๋๋ค. (default: `0.005`)

- `lr_scheduler`: LR ์ค์ผ์ค๋ฌ๋ฅผ ์ค์ ํฉ๋๋ค. ***Cosine Annealing LR Decay***(`'cosine'`)๋ฅผ ์ง์ํฉ๋๋ค. (default: `cosine`)

  

### Inference Phase

#### I. Singular Model Inference

```python
>>> python submit_singular --task 'main' --model-type 'VanillaEfficientNet' --transform-type 'random'
```

- `task` : ๋ฉ์ธ task(`'main'`), ๋ง์คํฌ ์ํ(`'mask'`), ์ฐ๋ น๋(`'ageg'`), ์ฐ๋ น(`'age'`), ์ฑ๋ณ(`'gender'`)์ 5๊ฐ์ง task์ ๋ํ ์ถ๋ก ์ด ๊ฐ๋ฅํฉ๋๋ค. (default: `'main'`)

- `model_type` : ๋ถ๋ฌ์ฌ ๋ชจ๋ธ ์ํคํ์ณ๋ฅผ ์ ํํฉ๋๋ค. ์ง์ํ๋ ๋ชจ๋ธ ์ํคํ์ณ๋ ***VanillaEfficientNet***(`'VanillaEfficientNet'`), ***VanillaResNet***(`'VanillaResNet'`), ***MultiLabelTHANet*** (`'MultiLabelTHANet'`), ***MultiClassTHANet***(`'MultiClassTHANet_MK1'`), ***THANet***(`'THANet_MK1'`)์ด ์์ต๋๋ค. (default: `'VanillaEfficientNet'`)

- `load_state_dict` : ์ถ๋ก ์ ํ์ฉํ  ์ฌ์  ํ์ต๋ ํ๋ผ๋ฏธํฐ ํ์ผ์ ๊ฒฝ๋ก๋ฅผ ์ค์ ํฉ๋๋ค. ๋ชจ๋ธ ์ํคํ์ณ์ ๋ง๋ ํ๋ผ๋ฏธํฐ ํ์ผ์ ๋ถ๋ฌ์์ผ ์ ์ ์๋ํฉ๋๋ค.

- `transform_type`: Augmentation์ ํ์ฉํ  Transform ์ข๋ฅ๋ฅผ ์๋ ฅํฉ๋๋ค. ***Base***(`'base'`), ***Random***(`'random'`), ***TTA***(`'tta'`), ***Face Crop***(`'facecrop'`)์ ์ง์ํ๋ฉฐ, ๊ฐ transform์ ๋ค์๊ณผ ๊ฐ์ต๋๋ค. (default: ***Base***(`'base'`))

- `data_root`: ์ถ๋ก ํ  ๋ฐ์ดํฐ์ ๊ฒฝ๋ก๋ฅผ ์๋ ฅํฉ๋๋ค. (default: `./input/data/images`)

- `save_path`: ์ถ๋ก  ๊ฒฐ๊ณผ๋ฅผ ์ ์ฅํ  ๊ฒฝ๋ก๋ฅผ ์๋ ฅํฉ๋๋ค. ์ถ๋ก  ๊ฒฐ๊ณผ๋ ImageID์ ans์ ๋ ์ปฌ๋ผ์ ํฌํจํ csv ํ์ผ ํํ๋ก ์ ์ฅ๋ฉ๋๋ค. (default: `'./prediction'`)

  

#### II. Ensemble Inference

```python
>>> python submit_ensemble --task 'main' --root './saved_ensemble_models' --transform-type --method 'soft' --top-k 3 --tta 2
```

- `task` : ๋ฉ์ธ task(`'main'`), ๋ง์คํฌ ์ํ(`'mask'`), ์ฐ๋ น๋(`'ageg'`), ์ฐ๋ น(`'age'`), ์ฑ๋ณ(`'gender'`)์ 5๊ฐ์ง task์ ๋ํ ์ถ๋ก ์ด ๊ฐ๋ฅํฉ๋๋ค. (default: `'main'`)

- `root` : ์์๋ธํ  ๋ชจ๋ธ์ด ์ ์ฅ๋ ํด๋ ๊ฒฝ๋ก๋ฅผ ์ค์ ํฉ๋๋ค. ํ์ฌ KFold ๊ธฐ๋ฐ์ ์์๋ธ๋ง์ด ์ง์๋๊ธฐ ๋๋ฌธ์, ์๋ ฅํ ๊ฒฝ๋ก๋ ๋ค์๊ณผ ๊ฐ์ ๋๋ ํ ๋ฆฌ ๊ตฌ์กฐ๋ฅผ ๊ฐ์ ธ์ผ ํฉ๋๋ค. K-Fold์ ์ต์์ ํด๋์ ์ด๋ฆ์ ๊ธฐ์ค์ผ๋ก ๋ชจ๋ธ ์ํคํ์ณ๋ฅผ ๋ถ๋ฌ์ค๊ธฐ ๋๋ฌธ์, ์ต์์ ํด๋์๋ ๋ชจ๋ธ๋ช์ด ๋ฐ๋์ ๊ธฐ์ฌ๋์ด์ผ ํฉ๋๋ค. 

  ```shell
  kfold-ensemble-VanillaEfficientNet
   โโfold00
   โโfold01
   โโfold02
   โโ ...
   โโfoldNN
  ```

- `transform_type`: Augmentation์ ํ์ฉํ  Transform ์ข๋ฅ๋ฅผ ์๋ ฅํฉ๋๋ค. ***Base***(`'base'`), ***Random***(`'random'`), ***TTA***(`'tta'`), ***Face Crop***(`'facecrop'`)์ ์ง์ํ๋ฉฐ, ๊ฐ transform์ ๋ค์๊ณผ ๊ฐ์ต๋๋ค. (default: ***Base***(`'base'`))

- `data_root`: ์ถ๋ก ํ  ๋ฐ์ดํฐ์ ๊ฒฝ๋ก๋ฅผ ์๋ ฅํฉ๋๋ค. (default: `'./input/data/images'`)

- `top_k`: ๊ฐ Fold๋ณ๋ก ๋ช ๊ฐ์ ๋ชจ๋ธ์ ์์๋ธ์ ํ์ฉํ  ์ง ์ค์ ํฉ๋๋ค. Fold๋ณ ๊ฒ์ฆ ์ฑ๋ฅ์ด ๊ฐ์ฅ ๋์๋ ๋ชจ๋ธ๋ถํฐ ๋ถ๋ฌ์ต๋๋ค. ๊ฐ๋ น, `top_k = 2`๋ก ์ค์ ํ  ๊ฒฝ์ฐ, ๊ฐ Fold๋ก๋ถํฐ ์ฑ๋ฅ์ด ๊ฐ์ฅ ์ข์ 2๊ฐ์ ๋ชจ๋ธ์ ๋ถ๋ฌ์ ์ด `(# of Folds) x 2` ๋ฒ์ ์ถ๋ก ์ ์งํ, ์์๋ธํฉ๋๋ค. (default: `3`)

- `method`: ์์๋ธ ๋ฐฉ์์ ์ค์ ํฉ๋๋ค. Hard Voting ๋ฐฉ์(`'hardvoting'`)๊ณผ Soft Voting ๋ฐฉ์(`'softvoting'`)์ด ์์ผ๋ฉฐ, Soft Voting ์ค์  ์ ์ฐ์ ํ๊ท (`arithmetic`), ๊ธฐํํ๊ท (`geometric`), ๊ฐ์คํ๊ท (`weighted_mean`)์ ํ์ฉํ ์ถ๋ก  ๊ฒฐ๊ณผ๋ฅผ ๋ชจ๋ ์ ์ฅํฉ๋๋ค. (default: `'softvoting'`)

- `save_path`: ์ถ๋ก  ๊ฒฐ๊ณผ๋ฅผ ์ ์ฅํ  ๊ฒฝ๋ก๋ฅผ ์ง์ ํฉ๋๋ค. (default: `'./prediction'`)

- `tta `: TTA(Test Time Augmentation) ๊ฐ์ ์ง์ ํฉ๋๋ค. ๊ฐ๋ น, `tta=2`๋ก ์ค์ ํ  ๊ฒฝ์ฐ, ์ถ๋ก  ๊ณผ์ ์์ ํ๋์ ์ด๋ฏธ์ง๋ฅผ 2๊ฐ๋ก Augmentation, 2๊ฐ์ ์ถ๋ก  ๊ฒฐ๊ณผ๋ฅผ ์์๋ธํ์ฌ ์ต์ข์ ์ธ ์ถ๋ก ์ ํ๊ฒ ๋ฉ๋๋ค. (default: `1`)



## Dataset

### I. Split

![data split](https://github.com/iloveslowfood/ImageClassfication/blob/main/etc/data%20split.png?raw=true)

์ฃผ์ด์ง ํ์ต ๋ฐ์ดํฐ ์ค 90%๋ฅผ ํ์ต์ฉ ๋ฐ์ดํฐ๋ก, ๋๋จธ์ง 10%๋ฅผ ๊ฒ์ฆ์ฉ ๋ฐ์ดํฐ๋ก ํ์ฉํ์ต๋๋ค. ํฉ๋ฆฌ์  ๊ฒ์ฆ์ ์ํด ๋ฐ์ดํฐ๋ฅผ ์ด๋ฏธ์ง ๋จ์๊ฐ ์๋ ์ฌ๋ ๋จ์๋ก ๋ถ๋ฆฌํ๋๋ฐ, ์ด๋ ์ด๋ฏธ์ง ๋จ์๋ก ๋ฐ์ดํฐ๋ฅผ ๋ถ๋ฆฌํ  ๊ฒฝ์ฐ ํน์  ์ฌ๋์ ์ด๋ฏธ์ง๊ฐ ํ์ต์ฉ ๋ฐ์ดํฐ์ ๊ฒ์ฆ์ฉ ๋ฐ์ดํฐ ๋ชจ๋์ ๋ฑ์ฅํด ๊ฒ์ฆ ๊ฒฐ๊ณผ๋ฅผ ์ ๋ขฐํ  ์ ์๋ data leakage ๋ฌธ์ ๊ฐ ๋ฐ์ํ  ์ ์๊ธฐ ๋๋ฌธ์๋๋ค. ๋ํ, ์ฃผ์ด์ง ํ์ต ๋ฐ์ดํฐ์ ๋ถํฌ๊ฐ public/private ๋ฐ์ดํฐ์ ๋ถํฌ์ ๊ฐ๋ค๋ ๊ฐ์  ํ์, ํ์ต์ฉ ๋ฐ์ดํฐ์ ๊ฒ์ฆ์ฉ ๋ฐ์ดํฐ์ ๋ถํฌ๊ฐ ๊ฐ๋๋ก ์ธตํ์ถ์ถ๋ฒ์ ํ์ฉํด ์ฃผ์ด์ง ๋ฐ์ดํฐ๋ฅผ ๋ถ๋ฆฌํ์๊ณ , ๊ฒ์ฆ์ฉ ๋ฐ์ดํฐ์๋ ์ด๋ ํ ๊ฐ๊ณต๋ ์ทจํ์ง ์์์ผ๋ก์จ ๊ฒ์ฆ ๊ฒฐ๊ณผ์ ์ ๋ขฐ์ฑ์ ํ๋ณดํ์ต๋๋ค.

### II. Oversampling

![mixup](https://github.com/iloveslowfood/ImageClassfication/blob/main/etc/mixup.png?raw=true)

์ฃผ์ด์ง ๋ฐ์ดํฐ๋ 18๊ฐ์ง ์นดํ๊ณ ๋ฆฌ๋ณ ๋ถ๊ท ํ์ด ์กด์ฌํฉ๋๋ค. ๋๋ฌธ์ ๋น๊ต์  ๋ถ์กฑํ ์นดํ๊ณ ๋ฆฌ์ ๋ฐ์ดํฐ๋ฅผ ์ค๋ฒ์ํ๋งํ ๋ฐ์ดํฐ์์ ์ถ๊ฐ ๊ตฌ์ฑ, ์ฃผ์ด์ง ๋ฐ์ดํฐ์ ๋๋ถ์ด ๋ชจ๋ธ ์คํ์ ํ์ฉํ์ต๋๋ค.

## Augmentation

### I. Base

CentorCrop ๋ฑ ๊ฐ์ฅ ์ผ๋ฐ์ ์ธ ์ด๋ฏธ์ง ๊ฐ๊ณต ๋ฐฉ๋ฒ์ผ๋ก ๊ตฌ์ฑ๋ Augmentation์๋๋ค. ([์์ค์ฝ๋ ๋ณด๊ธฐ](https://github.com/iloveslowfood/ImageClassfication/blob/a14c97f0d2253122a798913fbd29a7bdcb92f128/augmentation.py#L9))

### II. Random

์ด๋ฏธ์ง๋ฅผ ์์๋ก ๊ฐ๊ณตํ๋ ๋ฐฉ๋ฒ์ ํฌํจํ Augmentation์ผ๋ก, [RandAugment](https://github.com/ildoonet/pytorch-randaugment) ๋ชจ๋์ ํ์ฉํฉ๋๋ค.([์์ค์ฝ๋ ๋ณด๊ธฐ](https://github.com/iloveslowfood/ImageClassfication/blob/a14c97f0d2253122a798913fbd29a7bdcb92f128/augmentation.py#L102))

### III. TTA

TTA(Test Time Augmentation)์ ํ์ฉํ๊ธฐ ์ํ Augmentation์ผ๋ก, Train ๋จ๊ณ์์๋ โRandomโ Augmentation๊ณผ ๊ฐ์ Augmentation์ด ์งํ๋๊ณ , Inference ๋จ๊ณ์์๋ `RandomResizedCrop()`์ ๋ฌด์์์  Augmentation์ ํ์ฉํ๋ค๋ ํน์ง์ด ์์ต๋๋ค.([์์ค์ฝ๋ ๋ณด๊ธฐ](https://github.com/iloveslowfood/ImageClassfication/blob/a14c97f0d2253122a798913fbd29a7bdcb92f128/augmentation.py#L102))

### IV. Face Crop

ํฝ์์ ๋ถํฌ๋ฅผ ๋ฐํ์ผ๋ก ์ด๋ฏธ์ง ๋ด ์ผ๊ตด ๋ถ๋ถ์ cropํ๋ Augmentation์๋๋ค.([์์ค์ฝ๋ ๋ณด๊ธฐ](https://github.com/iloveslowfood/ImageClassfication/blob/a14c97f0d2253122a798913fbd29a7bdcb92f128/augmentation.py#L102))



## Models

### I. VanillaEfficientNet

![veffi](https://github.com/iloveslowfood/ImageClassfication/blob/main/etc/veffi.png?raw=true)

Pretrained EfficientNet(`'efficientnet-b3'`)์ Backbone์ผ๋ก ํ๋ ๊ฐ๋จํ ์ด๋ฏธ์ง ๋ถ๋ฅ ๋ชจ๋ธ์๋๋ค. ([์์ค์ฝ๋ ๋ณด๊ธฐ](https://github.com/iloveslowfood/ImageClassfication/blob/05f60efadc8865b5f76e9503881b5337e5d64313/model.py#L43))

### II. VanillaResNet

![vres](https://github.com/iloveslowfood/ImageClassfication/blob/main/etc/vres.png?raw=true)

Pretrained ResNet(`resnet50`)์ Backbone์ผ๋ก ํ๋ ๊ฐ๋จํ ์ด๋ฏธ์ง ๋ถ๋ฅ ๋ชจ๋ธ์๋๋ค. ([์์ค์ฝ๋ ๋ณด๊ธฐ](https://github.com/iloveslowfood/ImageClassfication/blob/05f60efadc8865b5f76e9503881b5337e5d64313/model.py#L70))

### III. MultiClassTHANet

![thanet](https://github.com/iloveslowfood/ImageClassfication/blob/main/etc/thanet.png?raw=true)

Pretrained Image Network์ Attention ์ํคํ์ณ๋ฅผ ํ์ฉํ ์ด๋ฏธ์ง ๋ถ๋ฅ ๋ชจ๋ธ์๋๋ค. ([์์ค์ฝ๋ ๋ณด๊ธฐ](https://github.com/iloveslowfood/ImageClassfication/blob/7ef05acccfa04a386a6b98a4e471e8572ea75ff2/model.py#L96))

### IV. MultiLabelTHANet

![thanet_ml](https://github.com/iloveslowfood/ImageClassfication/blob/main/etc/thanet_ml.png?raw=true)

Pretrained Image Network์ Attention ์ํคํ์ณ๋ฅผ ํ์ฉํ ์ด๋ฏธ์ง ๋ถ๋ฅ ๋ชจ๋ธ์๋๋ค. ([์์ค์ฝ๋ ๋ณด๊ธฐ](https://github.com/iloveslowfood/ImageClassfication/blob/7ef05acccfa04a386a6b98a4e471e8572ea75ff2/model.py#L171))

### V. THANet

![thanet_3](https://github.com/iloveslowfood/ImageClassfication/blob/main/etc/thanet_3.png?raw=true)

Pretrained Image Network๋ฅผ ํ์ฉํ Multi-label ์ด๋ฏธ์ง ๋ถ๋ฅ ๋ชจ๋ธ์๋๋ค. ([์์ค์ฝ๋ ๋ณด๊ธฐ](https://github.com/iloveslowfood/ImageClassfication/blob/7ef05acccfa04a386a6b98a4e471e8572ea75ff2/model.py#L240))