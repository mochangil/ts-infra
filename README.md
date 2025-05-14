# 맞춤형 초개인화 레시피 서비스

<br>

### **🔖 프로젝트 개요**
  - **주제** : 사용자가 보유한 재료와 선호도를 기반으로 레시피를 추천해주고, 상황에 맞는 레시피로 변형하여 제공해주는 서비스 
  - **대상** : 한정된 상황에서도 다양한 레시피를 이용해 요리하고 싶은 1인 가구

</br>



![1](https://github.com/user-attachments/assets/15670833-34e6-45c3-92cb-c0249c47f20b)


![7](https://github.com/user-attachments/assets/149daa3c-44f5-4883-bc4d-7add01666bbf)
![8](https://github.com/user-attachments/assets/33f9da01-d2dd-4f7f-99dc-ba9305e578f3)

### 🔗 주요 기능  

  #### 1️⃣ 식재료 관리
  - 내가 보유한 식재료를 등록하고 제거할 수 있습니다.
  - 보유한 식재료 현황을 알 수 있습니다.
  - 유통기한을 설정하여, 유통기한이 지난 식재료는 폐기 처리됩니다.

  #### 2️⃣ 레시피 추천
  - 신체 조건, 보유한 식재료를 기반으로 레시피를 추천해줍니다.
  - 여러 범주에서 선호하는 레시피 태그를 설정합니다.
    - 한식, 양식, 중식 등
    - 건강식, 고단백, 채식 등
    - 구이, 볶음, 찜, 튀김 등
  - 알레르기 식재료를 설정해 포함되지 않도록 합니다.

  #### 3️⃣ 나만의 레시피 생성
  - 추천받은 레시피, 혹은 선택한 레시피를 나만의 레시피로 변형합니다.
  - 저렴하게, 건강하게, 한식풍으로, 양식풍으로 등으로 레시피를 변형할 수 있습니다.
  - 변형된 레시피는 레시피 보관함에 저장됩니다.

  #### 4️⃣ 커뮤니티
  - 레시피 리스트에서 마음에 드는 레시피를 스크랩할 수 있습니다.
  - 카카오톡 공유하기 버튼을 눌러 친구 혹은 채팅방에 레시피 링크를 공유할 수 있습니다.

</br>

### 🌈 개선 사항

  #### 1️⃣ 클러스터 Scale-Out 속도 개선 - 새로운 Pod running 까지의 시간 48% 단축

  </br>

  **문제 상황**
  - 기존에는 Cluster Autoscaler(CAS) + Auto Scaling Group(ASG) 조합을 통해 노드 오토스케일링을 구성
  - 새로운 노드 생성 + 파드 스케줄링까지 평균 90초 이상 소요
  - 트래픽이 급증하는 이벤트 발생 시, 노드 생성 지연으로 인한 서비스 불안정성 우려

  **1. nodeAffinity 설정을 통한 워크로드 분리**
  - 워크로드 특성에 따라 미리 지정된 노드 스펙에만 스케줄되도록 nodeAffinity 설정
  - 새로운 노드 생성 방식이 CAS -> ASG -> EC2 API로 간접 호출하는 방식이기에 느림
  - kube-apiserver의 상태를 주기적으로 스캔(10s ~ 30s)해서 Unscheduled Pod(pending) 감지
  - scale out 상황은 덜 일어나지만, scale out 속도는 개선되지 않음.

  **2. Karpenter 도입**
  - 새로운 노드 생성 방식이 Karpenter -> EC2 API로 직접 호출하는 방식이기에 빠름
  - kube-apiserver에서 Pod 리소스를 watch하여 Unscheduled Pod(pending) 발생시 즉시 감지
  - EC2 API를 직접 호출하기에 scale out뿐 아니라 scale up / down도 가능
  - 비효율적인 노드를 자동 제거하여 비용 최적화 유도
    - NodePool CRD의 consolidationPolicy, consolidateAfter 설정

  **결과**
  
  <img width="727" alt="image" src="https://github.com/user-attachments/assets/625658b0-f268-4993-b6f3-7cbab5c1c4e9" />

  
  <img width="725" alt="image" src="https://github.com/user-attachments/assets/1d635b17-93bb-4f4b-89f2-1eccc5df7e99" />


</br>

  #### 2️⃣ App of Apps 패턴을 통한 ArgoCD UI 및 배포 관리 체계 개선

  </br>

  **문제 상황**
  - 기존에는 단일 ArgoCD Application으로 클러스터 전체 리소스를 관리
  - 공동 작업자의 특정 워크로드 배포가 전체 클러스터에 영향을 줄 수 있는 구조
  - UI 상에서 각 워크로드의 배포 상태를 명확하게 구분하거나 추적하기 어려워 관리가 비효율적

  **App of Apps 패턴 도입**
  - Parent Application이 여러 Child Application을 참조하는 App of Apps 구조로 전환
  - 각 워크로드(서비스, 미들웨어 등)를 독립 ArgoCD Application으로 구성하여 분리 관리
  - 기존 Helm기반 관리방식은 각 서비스별 kustomization 설정파일로 전환하여 구조를 명확하게 하고, 가독성을 향상
  - 공동 작업자의 작업 범위가 각 워크로드로 한정되어, 영향 범위 최소화 및 책임 분리 가능

  **결과**

  <img width="720" alt="image" src="https://github.com/user-attachments/assets/ccd052f1-e729-4a30-9622-d5639f375474" />

  <img width="724" alt="image" src="https://github.com/user-attachments/assets/d8b08b6b-f13e-47e8-af87-5463c2fb0919" />


</br>

</br>

### 🍿 결과물

</br>

![14](https://github.com/user-attachments/assets/a23de59e-c555-4412-aca1-e135de2365f9)
![15](https://github.com/user-attachments/assets/49ba8bdc-1527-4f2d-81ff-aeaf729d88f8)



