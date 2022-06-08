
# style transfer를 통한 domain adaption 


### **photo synthesis 팀 구성**


| 이름   |  구성   |                      역할                  |
|:---:|:---:|:-------------------------------------:| 
| 류한웅 |  팀장  | yolov5, cycleGAN, 데이터 정제 및 생성   | 
| 안희상 |  팀원  | Neural Style Transfer, 외부 데이터 수집, PPT 작성  | 
| 허준호 |  팀원  | yolov5, cycleGAN, 데이터 정제 및 생성, 결과 시각화  | 
| 양원규 |  팀원  | Neural Style Transfer, 데이터 분석, 결과 시각화 | 



### **Environments**

- Ubuntu 18.04 LTS
- Docker
- AMD Ryzen5 5600x
- Geforce RTX 3080 12GB / RAM 32GB
- Jupyter notebook


<br>


## 🗒️ Content

--- 

### □ **데이터 정보**

| | #Sequences | #Images | Multiple Cities | Multiple Weathers | Multiple Times of Day | Multiple Scene types |
|:---:|:---:|:----:| :---:|:---:|:---:|:---:|
| Kitti | 22 | 14,999|No|No|No|Yes| 
| vKitti | 5|2,612|No|Yes|No|Yes|
| BDD100K |  100,000|120,000,000|Yes|Yes|Yes|Yes|


### 사용된 원본 데이터셋
- **kitti** : 자율주행을 위한 현실 데이터셋
- **vkitti 2**  : 자율주행을 위한 합성 데이터셋
- **BDD100K** : kitti에는 없는 다양한 환경의 현실 데이터셋

### 학습에 사용된 데이터셋
- **BDD100K** : 환경별로 분류하여 CycleGAN모델 학습에 사용
- [**kitti+vkitti(Baseline)**](preparing_dataset/yolov5/vkitti_kitti_generator.ipynb) : kitti와 vkitti를 각각 사용
- [**kitti+vkitti+CycleGAN**](preparing_dataset/yolov5/cycleGAN_train_dataset_generator.ipynb) : Baseline의 환경적인 부분을 [CycleGAN으로 생성한 데이터](preparing_dataset/cycleGAN/cycleGAN_generator/cycleGAN_generator.ipynb)로 대치
- **kitti+vkitti+NST** : Baseline의 환경적인 부분을 [neural style transfer로 생성한 데이터](preparing_dataset/NeuralStyleTransfer/Neural_Style_Transfer.ipynb)로 대치

### 검증에 사용된 데이터셋
- **BDD100K** : BDD100K의 train 데이터셋을 가려진 객체의 수의 비율로 분류하여 사용



<br>


## 📒 진행 과정

---

## Overview

### □ Purpose
- 실제 환경의 데이터셋을 합성 데이터셋에 반영하여 **domain gap reduce**
![image/Untitled%201.png](./imgs/vkitti_bdd100k.png)

### □ Structure
```
한웅's flow chart img
```

- 사용 모델 : yolov5, cycleGAN, neural style transfer
- 모델 평가 및 지표 : occluded dataset, mAP@0.5


## Process

### □ 기업 과업 소개(문제 인식 및 도출)
![image](https://user-images.githubusercontent.com/96898057/172413781-cee22cb5-6a19-458c-bc17-2351cc194cfa.png)
- 대낮과 같은 일반적인 환경에서는 detecting 성능이 준수하지만, 자연 환경이 변하면 detecting 성능이 떨어짐
- Photo Sythesis 팀은 다양한 환경에서도 detecting 성능을 높히는 방법 대해 연구하였으며, 제한된 데이터를 image augmentation을 통해 다양한 환경으로 증강 시킨 후 학습하여 성능 개선을 이룸

### □ 원인 분석
```
yolov5 + kitti + vkitti(모든 환경) img
```

yolov5모델에서 [Kitti+vkitti데이터셋](preparing_dataset/yolov5/vkitti_kitti_generator.ipynb)으로 학습시킨 경우 가려진 객체를 어느정도 탐지<br>

<details>
  <summary>
    가려진 객체를 잘 검출하게 되는 요인
  </summary>
  <div markdown="1">

  - ### Bag of freebies
    bag of freebies는 Data augmentation, Loss function, Regularization 등 학습에 관여하는 요소로, training cost를 증가시켜서 정확도를 높이는 방법들을 의미한다.
    
  - ### Bag of Specials
    Bag of Specials는 architecture 관점에서의 기법들이 주를 이루고 post processing도 포함이 되어 있으며, 오로지 inference cost만 증가시켜서 정확도를 높이는 기법들을 의미한다.
    
  - ### Self-Adversarial Training
    input image에 FGSM과 같은 adversarial attack을 가해서 model이 예측하지 못하게 만든다. 그 후 perturbed image와 원래의 bounding box GT를 가지고 학습을 시키는 것을 Self-Adversarial Training이라 한다. 이 방식은 보통 정해진 adversarial attack에 robustness를 높이기 위해 진행하는 defense 방식인데, 이러한 기법을 통해 model이 detail한 부분에 더 집중하는 효과를 보고 있다.
    
  - ### Mosaic Augmentation
    각기 다른 4개의 image와 bounding box를 하나의 512x512 image로 합쳐주며, 당연히 image의 모양 변화에 따라bounding box GT 모양도 바뀌게 된다. 이를 통해 하나의 input으로 4개의 image를 배우는 효과를 얻을 수 있어 Batch Normalization의 statistics 계산에 좋은 영향을 줄 수 있다고 한다. 
    
    Mosaic Augmentation을 이용하면 기존 batch size가 4배로 커지는 것과 비슷한 효과를 볼 수 있어 작은 batch size를 사용해도 학습이 잘된다.
    
    또한, 4개의 image를 하나로 합치는 과정에서 자연스럽게 small object들이 많아지다 보니 small object를 학습에서 많이 배우게 되어 small object에 대한 성능이 높아지는 효과도 
    있는 것 같다.
  </div>
</details>
<br>

```
밝은날(탐지 잘됨) / fog,rain,overcast(탐지 안됨) img
```
하지만 학습 데이터셋에 없는 다른 환경에 대해서는 성능이 저조함을 발견하고,
다양한 환경에 대한 학습을 위해 데이터 증강 기법을 적용할 필요성을 느낌
<br>

### □ 해결 방안 수립
- ### Domain Apdaptaion 
  ㅁㄴㅇㄹ
- ### DR
  ㅁㄴㅇㄹ
- #### 앤드류응 data centric 
  ㅁㄴㅇㄹ

>기존 vkitti의 다양한 환경에서의 데이터셋이 현실 세계에서의 특징을 제대로 반영하지 못하여 학습 성능이 떨어진다고 판단하여, cycleGAN과 neural style transfer와 같은 모델을 사용하여 현실 세계의 특성을 합성 데이터셋에 적용시킬 수 있다면 보다 더 나은 학습 결과를 기대할 수 있을 것

### □ data augmentation

#### Cycle Gan 

![ezgif com-gif-maker (1)](https://user-images.githubusercontent.com/96898057/172393501-7a137de4-29d3-42ce-9de9-38e3a57fc517.gif)![ezgif com-gif-maker](./gifs/overcast_resized.gif)<br>
　　　　　　　　　　　　original  　　　　　　　　　　　　　　　　　　　　 overcast<br><br>
![fog](./gifs/foggy_resized.gif)![rain](./gifs/rain_resized.gif)<br>
　　　　　　　　　　　　　fog 　　　　　　　　　　　　　　　　　　　　　　　rain


<details>
  <summary>CycleGan 이란?</summary>
  <div markdown="1">

  </div>
</details>
<br>

#### Neural Style Transfer 

![normal 20](https://user-images.githubusercontent.com/96898057/172377408-ae27f769-2bb4-407e-8989-969a4f999ddc.gif)![rain neural20](https://user-images.githubusercontent.com/96898057/172378068-0e5d78ea-3d48-40c6-a3fa-9b89c6b123a4.gif)<br>
　　　　　　　　　　　　original 　　　　　　　　　　　　　　　　　　　　　　rain<br>


  
<details>
<summary>Neural Style Transfer란?</summary>
<div markdown="1">

  - Neural Style Transfer
  
  Neural Style Transfer는 타겟 이미지의 콘텐츠는 보존하면서, 참조 이미지의 스타일을 타깃 이미지에 적용하는 방식이다.
  
  input image가 contents image로 보일 뿐만 아니라, style image의 style이 그려지도록 각각을 혼합하는데 사용되는 최적화 기술이다.
  
  아래 예시 이미지를 가져와봤다.
  
  ![image](https://user-images.githubusercontent.com/96898057/172281003-6fe3d26d-4edb-4246-836e-e4620b422750.png)
  
  Neural Style Transfer의 원리는 2가지 다른 함수를 정의하는 것으로 하나는 어떻게 두 이미지의 콘텐츠가 차이나는지 설명하고(Lcontent), 다른 하나는 두 이미지의 스타일의 차이(Lstyle)를 설명한다.
  
 즉, 기본 input image, 일치시키고 싶은 contents image와 style image를 선택한 후 contents와 style 간의 차이를 역전파(backpropagation)로 최소화함으로써 기본 input image를 
 변환한다.
  
  ![image](https://user-images.githubusercontent.com/96898057/172290573-1f26783c-66f8-450d-a498-1812983f66fa.png)
  
  위의 이미지에서 보이는 바와 같이 content image와 style image가 존재하고, 우리가 생성할 이미지 x는 white noise부터 시작해서 content의 정보와 style의 정보를 합성해서 얻는다.  (*white noise란 noise를 주파수 도메인으로 변환시켰을 때 스펙트럼이 전부 상수인 경우)
  
  모두 동일하게 pretrained VGG network를 활용하며 이때의 학습은 VGG network가 아니라 input image x가 backdrop되면서 점차 변화하는 것을 의미한다.
 
 - 각 image들의 iteration, 크기, 가중치들을 설정해준다.
  
  ![image](https://user-images.githubusercontent.com/96898057/172288996-831c7ce7-5f61-40ae-a636-c85d5fb638d6.png)

 - Content와 Style 표현
  image의 content와 style을 표현을 얻기 위해, model안에 중간 layer들이 있다.
  
  이 중간 layer들은 feature map을 나타내는데 이는 깊어질수록 높이가 커지게 된다. 우리는 미리 학습된 이미지 분류 신경망인 VGG16 신경망을 사용한다.
  
  이 신경망의 중간 layer들은 이미지의 style과 content의 표현을 정의하는데 필요하다.(중간 layer들에서 input image의 해당 style 및 content가 목적에 맞춰지도록 시도)
  
  -중간 layer
  학습된 이미지의 분류 신경망의 중간 layer 출력값들이 style과 content를 어떻게 정의할까?
  
  높은 layer 단계에서, 이 현상은 신경망이 (신경망이 학습해 온)image 분류를 하기 위해서는 반드시 image를 이해해야 한다. 원본 image를 입력 pixel로 사용하고 원본 image pixel을 image 내 feature들의 복잡한 이해형태로 변형하는 방식으로 내부 표현을 설계한다.
  
  이는 CNN이 얼마나 잘 일반화 될 수 있는지에 대한 이유이기도 하다. CNN은 배경이나 다른 노이즈들에 영향을 받지 않는 class 내에 존재하는 불변성(invariances)을 포착하고, feature들을 정의할 수 있다.
  
  그러므로 원본 image가 입력되고 분류 label이 출력되는 구간 어딘가에서 model은 복잡한 feature 추출기로 작동한다. 따라서 중간 layer에 접근함으로써 input image의 content와 style을 설명할 수 있다.
  
  ![image](https://user-images.githubusercontent.com/96898057/172283697-24576635-e248-4999-8769-c1cb58677389.png)

  - model
  
  우리가 사용한 모델 vgg16은 ResNet,Inception과 비교해 상대적으로 간단한 모델인 덕분에 Style Transfer를 하기에 더 효과적이다.
  
  style과 content의 feature에 해당하는 중간 layer 접근을 위해, 케라스를 사용해 원하는 출력을 activation으로 model을 정의함으로써 해당 출력값을 얻을 수 있다.
  
  ![image](https://user-images.githubusercontent.com/96898057/172284517-801b71f3-0664-4260-b31c-d7b23cf466a6.png)

  - content loss
  
  content loss 함수는 실제로는 간단한데, 적용하고픈 content image와 기본 input image를 신경망으로 통과시킬 수 있다.
  이는 설계 model에서 중간 layer의 출력을 반환한다. 그런 다음 그저 이미지들 간의 중간 표현들 사이에 유클리드 거리(Euclidean distance)를 취한다.
  (여기서 유클리드 거리는 다차원 상의 두 점 사이의 거리를 계산하는 공식이다.)
  
  이러한 content 손실을 최소화 하기 위해 일반 방식으로 역전파(backpropagation)을 수행한다. 따라서 특정 layer(content_layer에 정의된)에서 원본 content image로 유사한
  반응을 생성할 때까지 초기 image를 변환시킨다.
  
  ![image](https://user-images.githubusercontent.com/96898057/172286249-a3b02b87-cd1c-45bb-b9c7-0ce53d32db88.png)
  
  -Style loss
  Style loss를 계산하는 것은 content loss에 비해 좀 더 어렵지만, 동일한 원칙을 따른다. 이번에는 신경망에 기본 input image와 style image를 입력으로 사용한다.
  
  기본 input image를 위한 style을 생성하려면, content image에서 기울기 하강(Gradient Descent)을 수행하여 원래 image의 style표현과 일치하는 image로 변환한다.
  
  style image의 feature 상관관계(correlation) map과 input image사이의 평균 제곱 거리(MSE)를 최소화함으로써 이 작업을 수행한다.
  
 ![image](https://user-images.githubusercontent.com/96898057/172287410-069bf2c3-bb66-4617-a3a9-fbccb6aff03d.png)

  -경사하강법
  손실을 최소화하도록 반복적으로 출력 이미지를 업데이트할 것이다. 신경망과 관련된 가중치를 업데이트를 하지 않고, 대신 손실을 최소화하기 위해 input image를 훈련시킨다.
  이를 위해서는 loss와 기울기를 어떻게 계산하는지 알아야 한다. content 및 style image를 load하는 기능을 할 작은 함수를 정의하여 신경망에 image들을 input으로 주고, 모델에서 content 및 style feature 표현을 출력한다.
  ![image](https://user-images.githubusercontent.com/96898057/172288676-094c8d9e-a5d2-46d8-ad08-8bf4708e3b38.png)

  이러한 일련의 과정들을 거쳐 image를 생성한다.
![image](https://user-images.githubusercontent.com/96898057/172288880-5fb82ea4-e951-41e0-91b4-0c518f7d27dd.png)



  
</div>
</details> 


<br>


### □ 검증

### validation dataset
- BDD100K의 train데이터셋 전부를 val dataset으로 사용하였습니다.
- 가려진 객체(car,van,truck)의 개수가 '0-5', '0-10', '0-전체'인 데이터셋으로 분리하여 검증했습니다.
- '0-5', '0-10', '0-전체'는 가려진 객체의 비율(1:1:1)로 나누어 결정하였습니다.

  <img src="./imgs/occluded_instance_amount.png" width="500" height="400">

### kitti + vkitti + cycleGAN mAP@0.5 result
|occlusion amount|baseline|ours(cycleGAN)|
|:--:|:--:|:--:|
|0~5|0.379|**0.413**|
|0~10|0.358|**0.387**|
|0~all|0.342|**0.365**|



### kitti + vkitti + Neural Style Transfer mAP@0.5 result
|occlusion amount|baseline|ours(NST)|
|:--:|:--:|:--:|
|0~5|0.379|ㅁㄴㅇㄹ|
|0~10|0.358|ㅁㄴㅇㄹ|
|0~all|0.342|ㅁㄴㅇㄹ|

> 합성데이터에 환경정보를 추가한 데이터셋의 가려진 객체 탐지에 있어서 성능 향상이 있음을 확인할 수 있습니다.

<br><br>

## reference 

[1] [Structured Domain Randomization: Bridging the Reality Gap by
Context-Aware Synthetic Data](https://arxiv.org/pdf/1810.10093.pdf)

[2] [Photorealistic Style Transfer via Wavelet Transforms](https://openaccess.thecvf.com/content_ICCV_2019/papers/Yoo_Photorealistic_Style_Transfer_via_Wavelet_Transforms_ICCV_2019_paper.pdf) 

[3] [Unpaired Image-to-Image Translation using Cycle-Consistent Adversarial Networks](https://arxiv.org/pdf/1703.10593.pdf)

[4] [KITTI : Object Scene Flow for Autonomous Vehicles](http://www.cvlibs.net/publications/Menze2015CVPR.pdf)

[5] [Virtual KITTI 2(Cabon, Yohann and Murray, Naila and Humenberger, Martin)](https://arxiv.org/pdf/2001.10773v1.pdf)

[6] [BDD100K : A Diverse Driving Dataset for Heterogeneous Multitask Learning](https://arxiv.org/pdf/1805.04687.pdf)


[7] https://github.com/sukkritsharmaofficial/NEURALFUSE

[8] https://github.com/junyanz/pytorch-CycleGAN-and-pix2pix

[9] https://github.com/ultralytics/yolov5
















<br>

### timestamp
<details>
<summary>5월 13일</summary>

<div markdown="1">
yolov5을 vkitti 2.0.3의 일부분의 데이터를 가지고 시험적으로 학습시켜보았습니다.

<details>
<summary>yolov5m모델을 백본 네트워크로 사용</summary>
<div markdown="1">

  [yolov5/yolov5m.yaml at master · ultralytics/yolov5](https://github.com/ultralytics/yolov5/blob/master/models/yolov5m.yaml)
  
  train.py  --img 1248 --batch 8 --epochs 300 --data '../datasets/vkitti2.0.3.yaml' --cfg 'models/yolov5m.yaml' --weights yolov5n.pt --name only_clone
  
  - input img size 640(default)에서 1248로 변경
  </div>
</details> 
<details>
<summary>batch size는 8로  300 epoch 학습</summary>
<div markdown="1"> 
아래는 train_batch 예시

  ![image](https://user-images.githubusercontent.com/96898057/172299810-61d05ac3-eb6c-4c85-867c-32e51beb2154.png)
  </div>
</details> 
  <details>
<summary>학습 결과</summary>
<div markdown="1"> 
 
  ![image](https://user-images.githubusercontent.com/96898057/172299767-ff7085d5-3b35-4e32-8b74-9934fead2775.png)
  </div>
</details> 
  <details>
<summary>detection 결과</summary>
<div markdown="1"> 
  
  ![image](https://user-images.githubusercontent.com/96898057/172299717-a9da4e66-6c31-4caa-89be-ce105f0400e3.png)
  </div>
</details> 
  
  <details>
<summary>이슈</summary>
<div markdown="1"> 
  - vkitti 2.0.3 dataset에서 같은 바운딩 박스 내에 다른 클래스 객체가 들어있는 경우
    
  - 확인된 경우는 가로5 세로4 바운딩 박스이다.
   
  - 우리는 바운딩 박스로만 학습을 하여 레이블이 중복되는 현상이 있어 중복되는 경우를 삭제하고 학습을 시키는 쪽으로 진행

  - 학습할때 5일정도의 시간이 걸린다.

  - yolov5의 agumentation의 종류를 알고 아이디어를 정해야할듯

  - 학습할때얼마만큼의크기를제한할것인가
    </div>
</details> 
 
  </div>
</details> 
  
 <details>
<summary>5월 16일</summary>
<div markdown="1">
  전체 데이터셋으로 학습 시 시간이 너무 오래 걸려 kitti를 clone한 데이터셋만 학습
  
 <details>
<summary>yolov5n모델을 백본 네트워크로 사용</summary>
<div markdown="1">

  [yolov5/yolov5n.yaml at master · ultralytics/yolov5](https://github.com/ultralytics/yolov5/blob/master/models/yolov5n.yaml)

python train.py  --img 1248 --batch 32 --epochs 300 --data '../datasets/vkitti2.0.3.yaml' --cfg 'models/yolov5n.yaml' --weights yolov5n.pt --name only_clone

- input img size 640(default)에서 1248로 변경
  </div>
</details> 
  
 <details>
<summary>batch size는 32로  300 epoch 학습</summary>
<div markdown="1">  
  
  아래는 train_batch 예시
  ![image](https://user-images.githubusercontent.com/96898057/172300272-a4254673-6761-4e96-86bd-d8e29bba7972.png)
  </div>
</details> 
  
 <details>
<summary>학습 결과</summary>
<div markdown="1">  
  
  ![image](https://user-images.githubusercontent.com/96898057/172300426-b069967b-2660-45ec-98c2-a7efc28370b8.png)
  </div>
</details>  
  
 <details>
<summary>detection 결과</summary>
<div markdown="1">  

  [test_result - Google Drive](https://drive.google.com/drive/folders/1sZngP_ysdRXxTWBZm32POl61barXxK8h?usp=sharing)
  </div>
</details> 
  
 <details>
<summary>이슈</summary>
<div markdown="1">
  - detection 시 confficence score가 상대적으로 낮아 yolov5l6 모델을 사용할 예정
   </div>
</details> 
  </div>
</details>
  
 <details>
<summary>5월 17일</summary>
<div markdown="1">

  ![image](https://user-images.githubusercontent.com/96898057/172300912-08ecf1cc-423e-4f5d-949f-b2291d1181d7.png)
  ![image](https://user-images.githubusercontent.com/96898057/172300955-882988ef-6d9f-490f-be7b-b4d48bb8e6fb.png)
  - yolov5l6 모델 사용 : yolov5x6모델은 하드웨어 메모리 부족으로 학습 불가
  
 <details>
<summary>yolov5l6모델을 백본 네트워크로 사용</summary>
<div markdown="1">

  [yolov5/yolov5l6.yaml at master · ultralytics/yolov5](https://github.com/ultralytics/yolov5/blob/master/models/hub/yolov5l6.yaml)

python train.py  --img 1280 --batch 8 --epochs 300 --data '../datasets/vkitti2.0.3.yaml' --cfg 'models/yolov5l6.yaml' --weights yolov5l6.pt --name only_clone_l6

- input img size 1280(default)로 사용
  
  </div>
</details>
  
 <details>
<summary>batch size는 8로  300 epoch 학습</summary>
<div markdown="1"> 
  -아래는 train_batch 예시

  ![image](https://user-images.githubusercontent.com/96898057/172301207-5cc99794-ad17-4121-b544-9e0f37f8a8fa.png)
  </div>
</details>
 <details>
<summary>학습 결과</summary>
<div markdown="1"> 

  ![image](https://user-images.githubusercontent.com/96898057/172301289-fee0d253-9457-42eb-ac54-d162977a4261.png)
  </div>
</details>
 <details>
<summary>detection 결과</summary>
<div markdown="1">

  [test_result_l6 - Google Drive](https://drive.google.com/drive/folders/1B98K2GRVtaei3moXf61urC8gPZn5YEz5)
  </div>
</details>
  </div>
</details>




