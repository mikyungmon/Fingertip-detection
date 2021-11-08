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

최근의 CNN 기반 접근 방식은 많은 object dectection에서 좋은 결과가 있었지만 RGB 비디오에서 직접 손가락 끝을 감지하는 것을 크기가 극히 작기 때문에 매우 어렵다.

손 영역을 분할과 손 중심 localization가 있다면 새로운 signature 함수인 **거리 가중 곡률 엔트로피(distance-weighted curvature entropy)** 를 사용하여 분할된 손에서 손가락 끝 위치를 찾을 수 있다.

이 함수는 손 중심에서 분할(segmentation)된 이진 손 영역의 각 윤곽점까지의 거리와 각 윤곽점에서의 곡률 엔트로피를 곱하여 얻을 수 있다. 

손가락 끝은 곡률이 높은 지점이며 손 중심에서 멀리 떨어져 있다. 따라서 정확한 detection을 위해 **1) 경계의 곡률  2) 중심점부터 경계의 거리** 를 중요한 요소로 갖는다.

1)  Curvature of Contour (경계의 곡률)

    분할된 손 이진 영상에서 추출한 평면 윤곽 곡선을 s라고 하며 Suzuki et al.에서 제안한 경계 추적 알고리즘을 사용했다. 

    윤곽 검색 알고리즘은 픽셀 연결을 사용하여 손 영역의 외부 경계를 추출한다(가장자리 또는 모서리가 닿는 경우에 두 픽셀이 연결됨).

    추출된 윤곽선 s는 다음 그림의 (a)와 같다.

    ![image](https://user-images.githubusercontent.com/66320010/140469705-b8a5a847-82e6-4091-bfd0-b0beba519e8b.png)

    s의 길이가 L이고 n개의 균일한 간격의 점에서 샘플링되면 샘플링 간격은 다음과 같다.

    ![image](https://user-images.githubusercontent.com/66320010/140469895-c41198be-56a3-460e-89f9-93d6a1de4e29.png)

    샘플링된 윤곽 곡선을 따라 있는 두 점 사이에서 접선 방향이 회전각도(α) 만큼 변경된다.

    곡률 k는 곡선을 따라 이동할 때 점선 방향의 변화이므로 회전각도 α와 샘플링간격 △s 사이의 비율로 근사할 수 있다.

    ![image](https://user-images.githubusercontent.com/66320010/140470177-4e4f7eda-3365-4970-b812-b00918433867.png)

    Feldman and Singh(2005)의 유도에 따라 곡률 엔트로피 u는 다음과 같이 근사될 수 있다.

    ![image](https://user-images.githubusercontent.com/66320010/140470372-b9f6abe7-64c6-4b72-96f7-3bba92975ed1.png)

    따라서 곡률 엔트로피 u가 곡률 k(s)에 국부적으로 비례하고, 스케일이 변하지 않는다는 것을 발견했다. 

    이를 통해 높은 곡률 지점, 즉 손가락 끝을 localization 할 수 있다.
    
2) Distance between Centroid and Contour (중심점과 경계의 거리)

    signature 함수의 두 번째 요소는 거리 함수 δ(s)로, 각 윤곽점과 손의 중심(Xc, Yc) 사이의 거리와 같다.
    
💡 요소 1)과 (2)에서의 성질을 이용하여 최종적으로 다음과 같은 함수를 정의했다.

윤곽 Ψ(s)의 signature 함수를 윤곽 u(κ(s))의 곡률 엔트로피와 거리 함수 δ(s)의 곱으로 정의한다.

![image](https://user-images.githubusercontent.com/66320010/140473385-ab8c607f-c4b5-4af1-9838-688a6cb63f73.png)

매개변수 γ는 signature 함수의 거리 항에 대한 가중치이다.

signature함수 Ψ(s)는 손가락 끝 위치에서 최대값을 갖는다. 

따라서 손 가락의 끝의 좌표 (Xf,Yf)는 다음과 같이 계산할 수 있다.

![image](https://user-images.githubusercontent.com/66320010/140473965-7f9bf67f-d9fa-43ab-bfc0-d8f9b914ec16.png)

#### ◾ 2.3 Fingertip tracking and character trajectory generation #### 

앞에서 손가락의 위치를 파악했고 이제는 손가락을 실시간으로 계속 추적하고 감지된 손가락 끝 위치에 의해 형성된 궤적에서 문자를 생성하는 것이다.

이를 위해 제스처를 시작하는 손이 감지된 프레임에서 시작하여 중지 기준에 도달할 때까지 연속 프레임에서 손가락 끝을 추적해야 한다.

손끝 감지 오류나 손 떨림으로 인한 노이즈를 제거하기 위해서는 궤적 평활화 알고리즘을 적용한다.

   - **2.3.1 손가락 끝 추적 ( fingertip tracking )** 
   
       연속 프레임에서 손의 감지 및 추적은 손가락 끝 감지 및 추적 성능에 필수적이다.

       실험에 따르면 모든 프레임에 대해 Faster R-CNN detector를 사용하면 계산 비용이 크고 실시간 성능보다 프레임 속도가 훨씬 낮아진다.

       따라서 **KCF 추적 알고리즘**이 손 영역 추적에 사용된다.
       
       tracker는 Faster R-CNN detector 출력으로 초기화되고 재초기화는 t프레임 간격으로 수행된다.
       
       이 경우는 손은 고경도 물체이기 때문에 오랜 시간 동안 추적하기 어렵기 때문에 재초기화가 필요하다.
       
       재초기화 간격 t를 50으로 하는 것은 추적 정확도와 프레임 속도 차이에서 최상의 절충안이라고 실험적으로 밝혀졌다.
       
       정리하면, 다음과 같은 순서로 손가락 끝 추적을 한다.
       
       1) tracker 초기화는 Faster R-CNN으로 한다.
       2) 50 프레임동안 KCF 추적 알고리즘으로 손 추적한다.
       3) 50 프레임마다 Faster R-CNN으로 재초기화한다.

   - **2.3.2 종료 기준 ( termination criterion )** 

       전통적인 온라인 필기의 경우 같이 펜업 동작이 없기 때문에 종료 구분 기준은 air writing 시스템에서 중요하다.
       
       해당 연구에서는 **감지된 손가락 끝의 속도**를 종료 기준으로 사용한다.
       
       문자를 쓰는 동안 손가락 끝의 속도가 쓰기가 완료되고 손가락 끝이 거의 정지할 때와 비교하여 더 높기 때문이다.
       
       한 프레임에 대한 손가락 끝의 변위 d는 **두 점 사이의 유클리드 거리**로 주어지고 속도 v는 다음 방정식과 같이 **변위와 프레임 속도 f(fps)의 곱**이다.
       
       ![image](https://user-images.githubusercontent.com/66320010/140478167-4fe7df94-a22c-4724-abe1-d7a4a979eb9a.png)
       
       속도함수의 값이 특정 임계값보다 작으면 종료 조건은 만족하는 것으로 판단한다. 
       
       실험 결과, 임계값은 40이 대부분의 사용자에게 최상의 결과를 제공한다고 한다.
       
   - **2.3.3 궤적 평활화 ( trajectory smoothing )**  

        손끝 감지 불량이나 손 떨림은 글자 궤적이 고르지 않거나 왜곡되어 인식 단게에서 글자를 오분류 할 수 있다.
        
        따라서 글자 궤적을 부드럽게 하는 것은 노이즈를 감쇠시키기 위한 필수적인 후처리 단계이다.
        
        T를 이전 단계에서 얻은 문자 궤적이라고 해보자.
        
        Ti지점과 선행지점 Ti-1 사이의 거리가 더 크면 궤적 지점 Ti를 인접한 두 지점 Ti-1과 Ti+1의 평균값으로 대체하는 간단한 반복 평활 알고리즘을 사용한다.
        
        즉, 현재 그린 궤적과 이전에 그린 궤적의 거리 차이가 특정 임계값보다 클 경우, 오차라고 판단하여 Ti-1과 Ti+1의 **평균값**으로 대체한다.
        
        하지만 완전히 다른 글자를 그리는 경우일 수 있으므로 두 궤적의 엔트로피 차이가 특정 임계값보다 작은 경우 같은 궤적이라고 판단하고 그렇지 않은 경우는 다른 궤적이라고 판단한다.
        
        임계값은 정확도와 속도 사이의 균형을 맞추기 위해 실험적으로 5로 선택되었다.
        
        ![image](https://user-images.githubusercontent.com/66320010/140479152-d0a366d4-c7dc-4b9e-89f8-5e461747b8a5.png)
        
        제안된 평활화 알고리즘은 상당히 낮은 계산 비용으로 다른 알고리즘, 즉 Ramer-Douglas-Peucker 알고리즘(Misra et al., 2017)에 비해 경쟁력 있는 결과를 제공한다.
       
       
#### ◾ 2.4 Character recognition ####

그린 궤적이 어떤 문자인지 인식하는 단계이다.

본 실험은 EMNIST 데이터셋으로 미리 훈련한 AlexNet을 사용한다.

AlexNet은 매우 짧은 추론 시간으로 상당히 높은 정확도를 제공하기 때문이다.
       
![image](https://user-images.githubusercontent.com/66320010/140480598-94fa5b6d-655e-4117-ad63-991178107a88.png)       

### 3. Conclusion ###

- 해당 논문은 웹캠 비디오를 입력으로 사용하여 mid-air finger writing 인식을 위한 새로운 프레임워크 제시

- air writing의 초기화를 위한 새로운 writing hand pose detection 알고리즘을 제안

- 강력한 손가락 끝 감지 및 추적을 위해 거리 가중 곡률 엔트로피라는 새로운 signature 함수를 사용

- 손가락 끝 속도 기반 종료 기준은 쓰기 제스처의 완료를 표시하는 구분 기호로 사용됨

- 제안된 air writing 인식 프레임워크는 HCI에서 가장 터치리스 텍스트 입력 인터페이스로 응용 프로그램을 찾을 수 있음

- air writing 인식 프레임워크에서 몇 가지 실패 사례 존재 

![image](https://user-images.githubusercontent.com/66320010/140481394-7dd9cb54-1085-4f42-80de-a5744e67feb4.png)

- 위의 그림에서 a는 손가락 끝 감지 및 추적 오류로 실패 발생한 경우이고, b는 손가락 끝 감지 및 추적이 불량한 경우

- 이는 손이 프레임 내부에 완전히 들어가지 않은 경우 손 detector 성능이 저하된다는 사실로 설명 가능

### 참고 ###

논문 링크 https://doi.org/10.1016/j.eswa.2019.06.034

https://eochodevlog.tistory.com/113














