---
title: "[AWS] Vpc + Alb + ECS Fargate"
categories: AWS
tag: [AWS, 실습]
toc: true
author_profile: false
sidebar:
 nav: "docs"
#search: false
---



이번 포스팅에서는 VPC 내부에 ALB+ECS Fargate를 만들어보자.



![fargate-1](https://user-images.githubusercontent.com/75375944/209613940-a7ff63a7-8c5a-4229-b36c-47b0a08e73a0.png)



    

<aside>
💡 VPC 구축은 앞의 포스팅 [Vpc + Alb + Ec2]을 참고하세요!

</aside>

    

    

# Target Group

Alb를 만들기 위해 Target Group(대상그룹)부터 만든다.

타겟그룹은 Ecs Fargate 이므로 IP 유형으로 해야한다.

![fargate-2](https://user-images.githubusercontent.com/75375944/209613938-e28f44e6-8ebf-4def-be71-bb075435f5f3.png)

![fargate-3](https://user-images.githubusercontent.com/75375944/209613937-941c4b50-51b1-47d4-b443-15cfb23558a4.png)

![fargate-4](https://user-images.githubusercontent.com/75375944/209613936-6ada7a8e-9201-4a21-99d9-8c7e2e8a35f4.png)

![fargate-5](https://user-images.githubusercontent.com/75375944/209613935-5f8770e8-9de4-4bad-afa1-7a8fd13b5763.png)

    

ECS생성시에 타겟대상을 잡아주는 과정이 있으므로 
여기서 따로 추가할 필요없다.

![fargate-6](https://user-images.githubusercontent.com/75375944/209613934-631b93ac-4246-407f-bd23-1b3d9adfdebc.png)

생성 완료.

    

    

## 보안그룹

이번 포스팅에 보안그룹에서 헷갈릴만한 두 개를 미리 만들어 놓자.

- ALB의 보안그룹
- ECS 서비스 생성 시 사용할 보안그룹

    

### Two Point

- ALB의 보안그룹 인바운드

![fargate-7](https://user-images.githubusercontent.com/75375944/209613931-c94e5537-311d-42b9-814d-43a1273b3227.png)

- ECS 서비스 생성 시 사용할 보안그룹

주의할 점이라고 하면은 ECS에서 task를 수행 시 이미지를 가져올 때 80 포트로 개방을 해놔야한다는 점이다. 본인은 Alb에서 오는 인바운드만 열면된다고 생각해서 왜 이미지가 안가져와지는가 하며 한참을 헤맸다. (관련 에러는 포스팅 하단에서 기록함)

![fargate-8](https://user-images.githubusercontent.com/75375944/209613928-ecf881b2-170d-4c74-94f4-7312917824c8.png)

    

    

## ALB

ALB를 만들어준다.

![fargate-9](https://user-images.githubusercontent.com/75375944/209613926-48a6b343-4b7f-47a2-95fc-231c04bc3d6f.png)

![fargate-10](https://user-images.githubusercontent.com/75375944/209613923-3ce36390-e0eb-46a1-8110-c599192d4155.png)

새로 생성한 ALB는 다음과 같다.

![fargate-11](https://user-images.githubusercontent.com/75375944/209613922-35a1ef6c-29b9-430d-95e9-94ae33656133.png)

    

    

## 인터넷 게이트웨이, Nat gateway

인터넷 게이트웨이를 간단하게 만들어주고 ECS에 사용할 VPC에 붙여주자.

![fargate-12](https://user-images.githubusercontent.com/75375944/209613921-414b6d48-441c-4ada-94c7-be8a57e7c9a8.png)

    

그리고 Nat gateway도 만들어준다.

- 퍼블릭 서브넷
- 탄력적 IP 할당

![fargate-13](https://user-images.githubusercontent.com/75375944/209613920-e5dc1997-b4ab-4972-8a84-4cc0cec41769.png)

    

    

## 라우팅테이블

내 라우팅 테이블은 Public, Private 두 가지로 나누어져있다.

처음에는 뭐하러 나누나 했지만, 인터넷 게이트웨이와 Nat Gateway를 하나의 라우팅테이블로 두면 인터넷 게이트웨이가 방화벽기능으로 차단한다고한다. 이번 과정에서 나누는 이유를 알게 되었다.

- Public Routing Table
  - 인터넷 게이트 웨이를 붙여준다.

![fargate-14](https://user-images.githubusercontent.com/75375944/209613917-c837c281-cf15-44f5-bd33-634f05ed3aba.png)

- Private Routing Table 
  - Nat gateway를 붙여준다.

![fargate-15](https://user-images.githubusercontent.com/75375944/209613915-213ea823-fcd5-4d99-bae4-5861e1abbfca.png)

    

    

## ECS

### 작업정의

ECS 작업 정의부터 만들어준다.

Fargate 선택

![fargate-16](https://user-images.githubusercontent.com/75375944/209613914-bba60a7a-318b-4640-8a75-568af20d6093.png)

    

테스크 역할에는 해당 컨테이너가 다른 Aws 서비스에 api요청할 때 사용하므로 이번 포스팅에는 필요없다.

![fargate-17](https://user-images.githubusercontent.com/75375944/209613913-ff465262-3751-497d-9550-58743e73fdc2.png)

    

다음과 같이 넣어준다. 작업 실행 역할은 기본 role로 넣었다.

이 역할은 EC2관련, ECS관련 정책들이 들어가있다.

![fargate-18](https://user-images.githubusercontent.com/75375944/209613911-3cf5813c-c6ac-43f0-b97b-56b9989c9431.png)

    

컨테이너의 이미지는 아파치 httpd 사용

![fargate-19](https://user-images.githubusercontent.com/75375944/209613910-fee2e8fc-36e7-453f-879c-71fbf0dce3de.png)

![fargate-20](https://user-images.githubusercontent.com/75375944/209613908-44a46868-5908-4804-907b-3353a579097b.png)

테스트페이지를 띄우기 위해 환경에서 진입점과 명령어를 입력해준다.

- 명령어 `/bin/sh -c "echo '<html> <head> <title>Amazon ECS Sample App</title> <style>body {margin-top: 40px; background-color: #333;} </style> </head><body> <div style=color:white;text-align:center> <h1>Amazon ECS Sample App</h1> <h2>Congratulations!</h2> <p>Your application is now running on a container in Amazon ECS.</p> </div></body></html>' > /usr/local/apache2/htdocs/index.html && httpd-foreground"`

    

그리고 별다른 작업없이 생성.

    

    

### 클러스터

클러스터 생성은 간단하다.

Fargate를 사용할 것이기 때문에 네트워크 전용 선택

![fargate-21](https://user-images.githubusercontent.com/75375944/209613905-dc369ecb-033f-4c5a-b5cd-46225515c853.png)

    

기존 VPC 사용할 것이므로 별도 VPC 생성 X

![fargate-22](https://user-images.githubusercontent.com/75375944/209613899-5a9750ce-62eb-469a-b730-328c3d4793c5.png)

    

    

다음 클러스터의 서비스를 생성해보자.

만들어 놓은 작업정의 선택

![fargate-23](https://user-images.githubusercontent.com/75375944/209613898-a35eccee-05ea-42f9-8fb0-93f8900c3d65.png)

    

작업 개수는 1개여도 상관없다 ( 추후 조정가능)

그래도 2개 넣어서 진행

![fargate-24](https://user-images.githubusercontent.com/75375944/209613896-929cc9e3-ba3e-44be-bc9c-60277caa0ad6.png)

    

배포옵션은 롤링업데이트 방식

![fargate-25](https://user-images.githubusercontent.com/75375944/209613893-2734f458-dfa2-4fec-aaa2-6c5578c3c6ae.png)

    

네트워크 구성에서 주의할점은 ECS가 Private 서브넷에 둘 경우 자동 할당 퍼블릭 IP를 Disabled 해준다. 로드밸런서를 타고 다닐꺼라 필요가 없다. 만약 Public 서브넷에 둘 경우는 Enabled로 설정한다.

이번에는 Private 서브넷에 둘 것이므로 Disabled해준다.

![fargate-26](https://user-images.githubusercontent.com/75375944/209613892-e9c34e7f-f9b7-4afb-8c3d-39a2c07a9b6a.png)

    

보안 그룹도 잘 확인해주자.

80포트로 개방되고, 로드밸런서에서 트래픽을 받을 수 있도록 인바운드가 설정되어있어야한다.

![fargate-27](https://user-images.githubusercontent.com/75375944/209613890-81b8c94e-5eb9-49a1-b8fe-9ab5119f0f29.png)

    

로드밸런서는 미리 생성해둔 ALB로 잡고

![fargate-28](https://user-images.githubusercontent.com/75375944/209613889-e87fab12-d8bb-4197-827a-906acb3a135c.png)

    

대상그룹도 앞서 미리 생성해둔 걸로 잡는다. 처음 대상그룹 만들 때 ip를 따로 안잡은 이유가 여기서 잡기 때문이다.

![fargate-29](https://user-images.githubusercontent.com/75375944/209613887-139a8be5-0784-4a49-b89f-671f505e3a75.png)

    

오토스케일링은 적용 X

![fargate-30](https://user-images.githubusercontent.com/75375944/209613883-bf183f71-790f-4be5-b847-54bad331cd52.png)

    

생성해준다.

![fargate-31](https://user-images.githubusercontent.com/75375944/209613882-b6c608ba-d45e-48c3-a643-80079cda48bc.png)

    

작업상태에서 프로비저닝 후 Running 상태가 되야한다.

만약 여기서 Pending상태가 지속되거나 Stop된 경우 로그를 확인해주자.

로그는 Stoped된 컨테이너의 작업번호를 클릭하고 해당 컨테이너를 보면 로그가 찍혀있다.

![fargate-32](https://user-images.githubusercontent.com/75375944/209613881-8886f62f-8f6a-4b7b-b0f0-109df83f8f34.png)

    

Running 상태인 것을 보았으면 타겟그룹에 들어가보면 알아서 잡혀있는 것을 볼 수 있다.

초록색으로 건강하다~!

![fargate-33](https://user-images.githubusercontent.com/75375944/209613944-69eef178-5023-4395-afbe-b3809cdb8e39.png)

    

ALB의 DNS NAME 으로 접속해보는 것으로 마무리한다.

![fargate-34](https://user-images.githubusercontent.com/75375944/209613943-cdcd4e00-9283-4a18-86ce-f51aa0b0152b.png)

    

끝.

    

    

    

### 에러 해결

![fargate-35](https://user-images.githubusercontent.com/75375944/209613942-3fe8976c-5ed6-4148-b81a-e099ffe28bb3.png)

Log

> cannotpullcontainererror: error response from daemon: Get ~ net/http request canceled while waiting for connection(Client Timeout exceeded while awaiting headers)

    

이 에러는 ECS에서 이미지가 제때 받아와지지 않아 발생하는 에러로 무한로딩이 일어나 타임아웃이 발생한 것 같다. 다음을 통해 해결할 수 있다.

1. 이미지 url 제대로 확인
2. ecs의 보안그룹
3. public 라우팅테이블의 라우팅에 인터넷 게이트웨이가 0.0.0.0으로 열려있는지 확인
4. private 라우팅테이블의 라우팅에 nat gateway가 0.0.0.0으로 열려있는지 확인

    

나는 2번의 Ecs에 사용한 보안그룹 인바운드에 로드밸런서만 들어오게 해놔서 이미지가 pull 하지 못해 발생한 경우다.

우여곡절끝에 http 80으로 인바운딩을 추가하여 해결할 수 있었다.
