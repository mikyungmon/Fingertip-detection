# Fingertip detection for air-writing # 

### 1. Introduction ### 

- 가상 현실(VR)과 증강 현실(AR)의 등장으로 전통적인 HCI접근 방식을 대체할 자연스러운 인간-컴퓨터 상호작용 시스템 개발의 필요성이 빠르게 증가하고 있다.

- 특히 손 제스처 기반 상호 작용을 하는 인터페이스는 다양한 응용분야에서 인기를 얻고 있다.

- 하지만 손동작 제스처만으로는 텍스트를 입력하기에 충분하지 않다. 이것은 보다 자연스러운 인간-컴퓨터 상호작용 접근으로 이어지는 터치 및 전기 기계적 입력 패널을 대체할 수 있는 touch-less air-writing 시스템의 개발을 필요로 한다.

- 공중에 손가락으로 제스처를 그리면 인식하는 연구는 사전에 많이 진행되었다. 하지만 대부분의 연구는 아래와 같은 단점이 존재한다.

    1️⃣ 적외선 및 컬러, 관성 센서를 손가락에 붙여 인식하는 방법 ➡ 하드웨어 장치를 사용하는 접근 방식은 사용자에게 많은 행동 제약을 부과하고, 자연스러운 사용자의 패턴을 바꿀 수 있다.
    
    2️⃣ 하드웨어 착용이 없는 시스템을 개발 ➡ 사용자에게 정교한 하드웨어는 높은 비용과  제한된 가용성으로 인해 실제 응용 프로그램에 적합하지 않다.
    
    3️⃣ egocentric(자기 중심적) RGB 비디오에서 CNN기반 프레임워크를 사용 ➡ egocentric(자기 중심적) 비디오를 캡처하려면 스마트 카메라 또는 혼합현실 헤드셋 등이 필요하지만 일부 사용자는 사용할 수 없다.

- 따라서 이러한 문제를 해결하기 위해 해당 논문은 다음과 같은 기여를 한다.

    1️⃣ 정확한 손 감지를 위해 Faster R-CNN 프레임워크 사용하여 air writing 초기화를 위한 writing hand pose 감지 알고리즘 제안
    
    2️⃣ 손을 분할하고 마지막으로 손의 기하학적 속성을 기반으로 올려진 손가락의 수를 계산
    
    3️⃣ 거리 가중 곡률 엔트로피(distance weighted curvature entropy)라는 새로운 signature 함수 사용하여 강력한 손가락 끝 감지 접근 방식을 제안
    
    4️⃣ 제스처의 완료를 표시하는 손가락 끝 속도 기반 종료 기준을 제안
    
### 2. Proposed Algorithm ###   

![image](https://user-images.githubusercontent.com/66320010/140318900-630c1f6c-a6a4-4131-80ea-e72703f857ea.png)

제안된 알고리즘은 위 그림과 같으며 다음 네 가지 구성요소로 나눌 수 있다.

1) 제스처를 시작하는 손 인식하기 ( writing hand pose detection )
2) 각 프레임에서 손가락 끝 인식하기 ( fingertip detection )
3) 손가락 끝 추적 및 문자 궤적 생성 ( fingertip tracking and character trajectory generation )
4) 글자 인식하기 ( recognition of air-writing characters )


#### ◾ 2.1 Writing hand pose detection #### 

가장 먼저 사용자가 제스처를 시작했는지 판단을 해야한다.

해당 논문에서는 제스처의 조건을 손가락을 하나만 올렸을 때로 정의한다.

제스처를 시작하는 손을 인식하고 올린 손가락의 수를 계산하여 쓰지 않는 손 포즈와 구별한다.

이를 위해 **손 감지, 손 영역 분리, 손 중심점 검출, 올린 손가락 수 계산** 을 한다.

- **2.1.1 손 감지 ( hand detection )**

    Faster R-CNN 프레임워크가 손 감지에 사용되었다. 
    
    중간결과인(CNN의 결과) feature map을  RPN(Region Proposal Network)에 대한 입력하여 객체라고 예상되는 부분의 정보를 얻는다.
    
    그 다음에는 각 object proposal에서 특징을 추출하기 위한 관심 영역(RoI) 풀링을 하여 특징을 추출한다.
    
    마지막으로 이를 R-CNN(Region-Based Convolutional Neural Network)에 입력하여 물체를 분류하고 Faster R-CNN에 연결한다.
    
- **2.1.2 손 영역 분리 ( hand region segmentation )**    

    ![image](https://user-images.githubusercontent.com/66320010/140322432-bf49071f-f0da-4776-8701-cb550ab194eb.png)
    
    손 영역 분리는 두 단계의 접근 방식을 사용하여 달성된다.
    
    **피부색 분리 / 배경 빼기**  => 최종 이진 손 이미지는 둘의 집계로 얻을 수 있다.
    
   ◾ 피부색 분리 ( skin segmentation )

  해당 연구에서는 피부 분리를 위해 YCbCr 컬러 공간에서 정적 피부색 모델을 기반으로 하는 피부색 필터링 접근 방식을 제안한다.

  피부 광도는 상당히 다양한 반면 피부색은 인종에 따라 상당히 다르지만 피부색은 피부 유형에 따라 작은 범위에 불과하다고 한다.

  **따라서 Y 구성요소는 이용하지 않고 Cb, Cr만 이용한다.**

  영상의 RGB값을 YCbCr로 변환하고 

  ![image](https://user-images.githubusercontent.com/66320010/140323694-4f0f844a-e83c-494e-af3c-f2973b993258.png)

  이진화 하기 위해 Cb, Cr 값의 범위를 정한다.

  ![image](https://user-images.githubusercontent.com/66320010/140323719-0d1ec2e6-fb14-4ac1-84c6-1d0d5faadc2e.png)
      
   ◾ 배경 분리 ( background segmentation ) 
   
   배경 분리는 앞 단계에서 손을 검출한 bounding box안에 피부색과 가까운 개체(손의 일부가 아님)를 제거하는데 사용된다.
   
   ![image](https://user-images.githubusercontent.com/66320010/140324617-7e2c8fc4-cb6e-42c4-a077-ece0eb3aafed.png)
   
   본 연구에서는 **GMM(Gaussian Mixture Model) 기반** 배경 분리 알고리즘을 사용한다. 
   
   GMM의 주요 장점은 실시간 처리에 도달할 수 있다는 것이다.
   
   ◾ 앞의 두 과정 연산 ( combine operation ) 
   
   두 개의 분리(분할) 알고리즘의 집계(AND operation)는 손 bounding box 내부의 다른 물체로 인한 노이즈가 없는 정확한 손 분리(분할) 결과를 제공한다.
   
   또한 중간에 생기는 전경 객체(손)의 공간을 채우기 위한 **팽창** 연산, 노이즈를 제거하기 위한 **침식** 연산을 진행한다.
   
   ![image](https://user-images.githubusercontent.com/66320010/140325790-927d9d83-e1d2-41df-93e9-3d28ccfb4a61.png)
   
   위 수식에서 Ih1은 피부색 분리 결과 이미지, Ih2는 배경 분리 결과 이미지이다.
   
   ^는 AND 연산, +는 팽창 연산, -는 침식 연산을 의미한다.
   
   Ihr는 최종 손 이진 이미지 결과이다.
   
 - **2.1.3 손 중심점 검출 ( hand centroid localization )**     
   
    손 중심점은 손가락을 알아내기 위한 중요한 정보이다.
    
    손 중심의 초기 추정치를 찾기 위해 **두 가지 알고리즘**을 사용하고 **최종 중심은 둘의 평균**으로 계산한다.
    
    첫 번째 알고리즘은 **거리 변환 방법**이다.
    
    각 픽셀의 가장 가까운 경계점으로부터 거기로 표시된다. 아래 그림의 오른쪽 그림에서 밝은 부분일수록 경계점으로부터 거리가 멀다는 것을 의미한다.
    
    ![image](https://user-images.githubusercontent.com/66320010/140327331-445cf7d8-cace-4639-8923-015eba86f2f5.png)
    
    두 번째 알고리즘은 **이미지 영역 모멘트(image region moments)** 개념을 사용하는 것이다.
    
    픽셀 강도가 I(x, y)인 이미지에 대한 이미지 모멘트 Mij는 다음과 같이 계산할 수 있다.
    
    ![image](https://user-images.githubusercontent.com/66320010/140327707-85c2ba30-154a-4405-815e-c61cba5eb452.png)
    
    그리고 중심은 다음과 같이 구할 수 있다.
    
    ![image](https://user-images.githubusercontent.com/66320010/140327802-6fd49794-edd4-43a8-a80c-99f37ee9988e.png)
    
    여기서 M10, M01, M00은 이전 분할 단계에서 얻은 이진 손 영역의 모멘트이다.
    
    **최종 중심(xc, yc)은 두 개의 초기 추정값(두 개의 알고리즘으로 추정된 중심들)의 평균으로 구한다.**
    
- **2.1.3 올린 손가락 수 계산 ( counting raised fingers )**

    손의 기하학적 특성을 사용하여 올린 손가락의 수를 계산한다.
    
    ![image](https://user-images.githubusercontent.com/66320010/140329112-7a585b7b-a652-4199-9f4a-83afca4680f3.png)
    
    손의 중심(xc, yc)에서 최대 반경의 원을 그린다. 이 원을 최대 반지름을 갖는 내부 원이라고 부르고 반지름을 r로 취한다.
    
    또한 손의 중심에서 모든 올린 손가락을 교차하기 위해 반지름을 R로 갖는 원을 그린다.
    
    기존의 진행된 연구에 따르면  R = 1.5 x r 이라는 연구 결과가 있다.
    
    따라서 이를 이용하여 반지름을 R로 갖는 원의 영역을 구할 수 있다.
    
    큰 원에서 작은 원의 영역을 빼면 손가락 부분 하나, 손목 부분 하나가 남는다.
    
    해당 연구에서는 전경(foreground)과 배경(background)의 양방향 교집합(intersection)을 이용한다. 
    
    그렇기 때문에 손가락 부분 2개, 손목 부분 2개가 나온다. 이러한 교집합의 수를 n이라고 하자.
    
    최종적인 손가락 개수는 손목을 제외해야하므로 아래 수식을 이용한다.
    
    ![image](https://user-images.githubusercontent.com/66320010/140328991-71270fec-43ca-43b7-b57e-5db3a961c750.png)
    
    N이 1이면 제스처를 시작한다고 판단하고, air writing이 초기화된다.
    
#### ◾ 2.2 Fingertip detection ####     
