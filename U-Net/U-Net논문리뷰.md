## U-Net : Convolutional Network for Biomedical Image Segmentation

참고 : https://modulabs-biomedical.github.io/U_Net

논문 : https://arxiv.org/pdf/1505.04597.pdf

바이오메디컬랩@모두의연구소 - U-Net

![U net에 대한 이미지 검색결과](https://lmb.informatik.uni-freiburg.de/people/ronneber/u-net/u-net-architecture.png)

#### 그 자체로 모델의 형태가 U자로 되어 있어서 생긴 이름 : U-Net

### Abstract

* 전체적인 U-Net의 핵심 구조, data augmentation, ISBI challenge 에서 좋은 성능을 보였다.

* Data augmentation을 잘 활용하여 annotated sample을 보다 효율적으로 사용하는 training 전략을 보여줍니다.

* 논문에서 제안하는 아키텍처는 contracting path 에서는 context를 캡처하고, 대칭적인 구조를 이루는 expanding path에서는 정교한 localization을 가능하게 하는 구조입니다.


### 1 . Instroduction

* 논문에서 처음 소개하는 내용은 지난 2년동안(U-Net은 2015년 5월에 발표) deep convolution network는 많은 visual recognition 작업에서 매우 좋은 성능을 보였지만, training set의 크기와 고려할 네트워크의 크기 때문에 그 성공은 제한적이었다고 말하고 있습니다.
* Convolution Network의 일반적인 용도는 이미지에 대한 출력이 단일 클래스 레이블인 분류 작업에 있지만, 많은 시각 작업, 특히  biomedical processing에서 원하는 출력은 localization을 포함해야 하며, 즉 클래스 라벨은 각 pixel에 할당되어야 한다고 합니다.
* U-Net의 핵심
  * Convolution Encoder에 해당하는 Contracting Path + Convolution Decoder에 해당하는 Expanding Path의 구조로 구성. (해당 구조는 Fully Convolution + Deconvolution 구조의 조합)
  * Expanding Path 에서 Upsampling 할 때, 좀 더 정확한 Loacalization을 하기 위해서 Contracting Path 의 Feature를 Copy and Crop하여 Concat하는 구조.
  * Data Augmentation
  * 기존에는 Sliding-Window을 하면서 로컬 영역(패치)을 입력으로 제공해서 각 픽셀의 클래스 레이블을 예측했지만, 이 방법은 2가지 단점으로 인해서 Fully Convolution Network구조를 제안하고 있습니다.
* 두가지 단점
  * 네트워크가 각 패치에 대해 개별적으로 실행되어야 하고, 패치가 겹쳐 중복성이 많기 때문에 상당히 느리다.
  * Localization과 Context사이에는 Trade-off가 있는데, 이는 큰 사이즈의 patches는 많은 max-pooling을 해야해서 localization의 정확도가 떨어질 수 있고, 반면 작은 사이즈의 patches는 협소한 context만을 볼 수 있기 때문입니다.
* Contracting Path에서 Pooling되기 전의 Feature들은 Upsampling 시에 Layer와 결합되어 고해상도 output을 만들어 낼 수 있습니다.
* 하나 더 중요한 점은, 많은 수의 Feature Channels를 사용하는데, 아래 네트워크 아키텍처를 보면, DownSampling시에는 64채널 -> 1024채널까지 증가되고, Upsampling시에는 1024채널 -> 64채널을 사용하고 있습니다.
* 네트워크는 Fully Connected Layers를 전혀 사용하지 않고, 각 Layer에서 Convolution만 사용합니다.
* 다음으로 U-Net에서는 Segmentation시 Overlab-tile 전략을 사용합니다.

![U net fig 2에 대한 이미지 검색결과](http://openresearch.ai/uploads/default/original/1X/9db500ba287c18df96388b6250d9e6a571c0759b.jpg)

* Overlap-tile 전략은, U-Net에서 다루는 전자 현미경 데이터의 특성 상 이미지 사이즈의 크기가 상당히 크기 때문에 Patch 단위로 잘라서 Input으로 넣고 있습니다.
* 이 때, Fig.2 에서 보는 것과 같이 Border 부분에 정보가 없는 빈 부분을 0으로 채우거나, 주변의 값들로 채우거나 이런 방법이 아닌 Mirroring 방법으로 pixel의 값을 채워주는 방법입니다
* 노란색 영역이 실제 세그멘테이션 될 영역이고 , 파랑색 부분이 Patch입니다.
* 그림을 확대해서 자세히 보시면, 거울처럼 반사되어 Border부분이 채워진 것을 확인 할 수 있었습니다.
* Overlap-tile이라는 이름은, 파랑색 부분이 Patch 단위로 잘라서 세그멘테이션을 하게 되는데 이 부분이 아래 그림처럼 겹쳐서 뜯어내서 학습시키기 때문인 것 같습니다.

### 2 . Network Architecture

![U net에 대한 이미지 검색결과](https://lmb.informatik.uni-freiburg.de/people/ronneber/u-net/u-net-architecture.png)



Contracting Path는

1. 전형적인 Convolution network이고,
2. 두 번의 3X3 Convolution을 반복 수행하며 (unpadded convolution를 사용)
3. ReLU를 사용합니다
4. 2X2 max pooling과 stride 2를 사용함
5. downsampling시에는 2배의 feature channel을 사용하고



Expanding Path는

1. 2X2 convolution (up-convolution)을 사용하고,
2. feature channel을 반으로 줄여 사용합니다.
3. Contracting Path에서 Max-Pooling 되기 전의 feature map을 Crop하여 Up-Convolution 할 때 concatenation을 합니다.
4. 두번의 3X3 convolution 반복하며
5. ReLU를 사용합니다



마지막 Final Layer	에서는 1X1 convolution을 사용하여 2개의 클래스로 분류합니다.

U-Net은 총 23개의 convolution Layer가 사용됐습니다.



### 3 . Training

학습은 Stochastic gradient descent로 구현되었습니다.



이 논문에서는 학습시에 GPU memory의 사용량을 최대화 시키기위해서 batch size를 크게해서 학습시키는 것 보다 input tile의 size를 크게 주는 방법을 사용하는 데요.

이 방법으로 Batch size 가 작기 때문에, 이를 보완하고자 momentum의 값을 0.99값을 줘서 과거의 값들을 더 많이 반영하게 하여 학습이 더 잘 되도록 하였습니다.

![U net fig 3에 대한 이미지 검색결과](https://modulabs-biomedical.github.io/assets/images/posts/2018-04-02-U_Net/u-net_fig_3.png)



### softmax

$$
p_{k}(x) = exp(a_{k}(x))/\left(\sum_{k'=1}^Kexp(a_{k'}(x))\right)
$$



### Cross Entropy Loss with w(x)

각각 정답 픽셀에 대한 cross entropy loss에는 w(x)라는 가중치 값이 추가됩니다.

여기서
$$
P_{l(x)}의 ~~l(x)
$$
는 정답 클래스 즉 위 softmax 수식에서 정답의 레이블에 해당하는 k 값을 반환하는 함수입니다.



Cross entropy 함수는 정답의 추정값을 log에 사용하기 때문에 이에 해당하는 정답의 확률을 가져오는 것이죠. 수식 (1)은 loss값에 가중치 w(x)를 곱한 형태이며 이제 우리가 살펴볼 것은 w(x)입니다.


$$
E = \sum_{x\in\Omega}w(x)log(p_{l(x)}(x))   ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~(1)
$$

### w(x) 구하는 법 :

$$
w(x) = w_{c}(x)+w_{0}*exp\left(-\frac{(d_{1}(x)+d_{2}(x))^2}{2\sigma^2}\right)~~~~~~~~~~~~~~~~(2)
$$

Wc(x)는 x의 위치에 해당하는 클래스의 빈도수에 따라 값이 결정됩니다.

즉, 학습 데이터에서 x픽셀이 background일 경우가 많은지 foreground일 경우가 많은지의 빈도 수에 따라 결정된다고 보시면 됩니다.

그 뒤의 exp 텀은 d1,d2 함수를 포함하는데 d1은 x에서 가장 가까운 세포까지의 거리이고 d2는 두번째로 가까운 세포까지의 거리를 계산하는 함수입니다.

즉, x는 세포사이에 존재하는 픽셀이며 두 세포사이의 간격이 좁을 수록 weight를 큰값으로 두 세포사이가 넓을 수록 weight를 작은 값으로 갖게 됩니다. 이는 그림3(d)를 보시면 명확하게 확인할 수 있습니다. 

네트워크 파라미터 초기화는 He초기화 방법을 적용하였습니다.



### 3 . 1 Data Augmentation

Data Augmentation은 3 by 3 elastic 변환 행렬을 통해 수행합니다. 세포를 세그멘테이션 하는것이기 때문에 elastic deformation의 적용이 성능향상에 매우 큰 역할을 했다고 합니다.



### 4 . Experiments

학습한 이후 성능 지표는 EM Segmentation challenge에서 wraping Error / Rand Error / Pixel Error로 1위를 한 지표를 볼 수 있었습니다.

![U net table 1에 대한 이미지 검색결과](data:image/png;base64,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)

IOU(Intersection over union) 방법으로 측정한 결과로는 아래와 같이 92% / 77.5% 로 가장 좋은 성능을 보인 것을 확인 할 수 있었습니다.

![U net table 2에 대한 이미지 검색결과](data:image/png;base64,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)



### 5 . Conclusion

U-Net 구조는 매우 다른 biomedical segmentation applications에서 좋은 성능을 보였고, 이 성능을 보일 수 있었던 것은 Elastic 변환을 적용한 data augmentation 덕분이고, 이것은 annotated image가 별로 없는 상황에서 매우 합리적이었다고 합니다.

학습하는데 걸렸던 시간은 NVidia Titan GPU(6GB)를 사용했을 때, 10시간이었습니다.

이 U-Net 구현은 Caffe 기반으로 제공되고 있으며, U-Net의 아키텍처는 다양한 task에 쉽게 적용되서 사용될 것을 확신한다고 하면서 논문은 마무리됩니다.



Image Segmenation Task에서 가장 많이 쓰이는 U-Net은 U자형 아키텍처와 Fully Convolution & Deconvolution 구조를 가지고 있는 것으로 좀 더 정확한 Localization을 위해서 Contracting Path 의 Feature를 Copy and Crop해서 Expanding Path와 Concat하여 Upsampling을 한다는 것을 확실하게 알아두면 좋을 것 같습니다.