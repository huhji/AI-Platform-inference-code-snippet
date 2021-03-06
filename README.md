# ML inference code snippet with kubeflow

<center><img src="https://user-images.githubusercontent.com/59910975/136140737-3875ed24-af2b-4707-8526-7d7f8d3431dc.png" width="80%" height="80%"></center>

* 본 레포의 코드는 전체 프로세스의 일부만 떼어내 리팩토링한 것으로, 전체 Platform은 동작하지 않음을 밝힙니다.
* AI Platform은 위와 같은 프로세스로 진행됩니다.
* backend는 Django로 작성되었습니다.
* Kubeflow API를 통해 사용자의 요청에 따라 Kubernetes Pod를 생성합니다.


## 프로젝트의 목적
* 사용자가 코드 없이 클릭만으로 인공지능을 학습하는 플랫폼 개발을 목표로 합니다.
* 전문적인 도메인 데이터를 가진 비개발자를 타겟으로 합니다. 
* 인공지능 학습의 진입장벽을 낮추는 것에 기여합니다.

## 사용한 기술
* Django
* Kubeflow
* SQL
* Docker
* Kubernetes

## 디렉토리 구조 
```shell
.
├─aip_infer
│  │  admin.py
│  │  apps.py
│  │  service.py
│  │  tests.py
│  │  urls.py
│  │  views.py
│  │
│  └─services
│     InferenceService.py
│     StatusService.py
│
└─aip_main
    │  mlflow_models.py
    │  urls.py
    │
    └─daos
        inference_dao.py
        inference_log_dao.py
        preset_dao.py
        resource_status_dao.py

manage.py
README.md

```

## 진행 중 고려한 사항
* **사용자 편의성**
  * 비개발자인 타겟 입장에서 추론 과정을 직관적으로 이해할 수 있도록 화면을 구성하였습니다. (Kakao Vision API 화면 참고)
  * 추론 결과를 실시간으로 확인할 수 있도록 빠른 추론 연산을 목표로 개발하였습니다.
  
* **타 모듈과의 연계**
  * 추론 모듈은 프로세스의 특성상 데이터 모듈과 annotation 모듈, 학습 모듈과 연계성이 높습니다.
  * 각 모듈 담당자와 비즈니스 로직상 발생 가능한 여러 파이프라인 케이스를 놓고 꾸준히 커뮤니케이션하며 진행하였습니다.

* **효율적인 데이터, 모델 서빙**
  * 백엔드서버에서 소요되는 연산량과 메모리 점유율을 최대한 줄이기 위해 데이터나 모델 연산은 전부 Kubernetes Pod에서만 수행하도록 구현하였습니다.
  
* **정형 데이터의 처리**
  * 정형 데이터(Tabular data)의 추론은 학습에 사용한 형태와 동일해야 했기 때문에 어려움을 느꼈던 부분입니다.
  * 가령, 사용자가 전처리 이전의 test data를 입력한 경우 학습에 입력한 데이터와 동일한 순서로 전처리를 진행해야 했습니다.
  * 따라서 사용자가 학습 데이터와 형태가 다른 데이터를 추론에 입력했을 때 
    * ① 이 데이터가 학습 데이터와 동일한가 여부와
    * ② 전처리를 진행했을 때 동일해지는가 여부를 Pod 생성 요청 전 확인하였습니다.
  * 결과적으로, 사용자가 입력할 수 있는 모든 케이스에 대한 패턴을 정의하고 이를 단계별 코드로 구현하여 어떤 형식의 데이터를 입력해도 대응 가능한 코드를 작성하였습니다.
  
* **다양한 패턴의 테스트 케이스 구성**
  * 해당 플랫폼에서는 40여개의 인공지능 모델을 사용할 수 있습니다.
  * 각 모델에 입력하는 데이터의 양상도 상이했기 때문에 하나의 모델에 대해 다양한 경우의 테스트케이스를 각각 준비하여 테스트를 진행하였습니다.
