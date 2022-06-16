# 팀 Photo synthesis
### [허준호, 류한웅, 안희상, 양원규]
development by Photo synthesis


## 기업 과업 소개
<img width="1029" alt="image" src="https://user-images.githubusercontent.com/70171637/166200515-ed453473-008d-4ea7-9bbc-e37e7e92dd51.png">
- 실제데이터는 거리별 Blur 효과처럼 흐릿해지지만 합성데이터(vkitti)에서는 거리별, 자연 환경에 상관없이 선명하게 나타남
- Photo Sythesis 팀은 가상환경 내 거리별 차이와 다양한 환경에 대해 구현중에 있으며 이번 과제에서는 별도 OpenCV와 같은 영상처리 기법으로 데이터 증강 또는 같이 주어지는 Depthmap or Segmentation 이미지 정보를 통해 새로운 아이디어 제시 가능

--------------------------
## 원인 분석


현실세계에서 Detecting을 하는데 발생하는 원인

- 물체나 사물에 가려져 있는 객체
- 다양한 자연 환경으로 인한 blur로 Detecting 성능 저하
- 한정되어 있는 합성 데이터를 통해 학습할 시 다양한 환경에 대한 대비가 미흡
=> 맑은 날씨 기준의 데이터를 다양한 환경으로 변화 시켜 Object Detecting의 성능을 개선해보자


----------------------------------------
## 문제를 해결하기 위한 방안   

2. 자율 주행중 occlusion된 차량을 detecting 
- yolo v5 l6 모델을 사용
- CycleGan, Neural Style Transfer로 DataAugmentation을 하여 학습데이터양을 증대

<details>
<summary>사용한 모델 설명</summary>
<div markdown="1">


<details>
<summary>yolo v5 l6란?</summary>
<div markdown="1">
</div>
</details>


<details>
<summary>CycleGan 이란?</summary>
<div markdown="1">
</div>
</details>

  
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

  
  
  <코드 깃헙 url>
  
</div>
</details> 

  </div>
</details>





---------------------------------------------------------------------------------
2. Dataset
  
  -vkitti_2.0.3(vkitti_rgb(14Gb))
  
  -kitti_segmentation_map(1Gb)
  
  -kitti_rgb_1.3.1(15Gb)
  
  -kitti_rgb_2.0.3(8Gb)
  
  -kitti_vkitti_result_bdd
  
---------------------------------------------------------------------------------
3. 진행도

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
---------------------------------------------------------------------------------
  
예상 되는 문제점과 더 생각해볼점
- Neural Style Transfer를 사용하면서 어떻게 시간을 단축시킬 수 있을지
- 어떻게 더 자연스러운 환경 이미지를 구축할 수 있을지
- 
-
-
-
-
- 합성데이터의 자동차에만 blur 정도를 적용시켜 DR 효과를 줄 수 있을까?

-------------------------------------------------------------
# reference 

[1] Photorealistic Style Transfer via Wavelet Transforms(Jaejun Yoo, Youngjung Uh, Sanghyuk Chun, Byeongkyu Kang, Jung-Woo Ha)  

[2] https://github.com/sukkritsharmaofficial/NEURALFUSE

[3]
