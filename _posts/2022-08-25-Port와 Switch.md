---
layout: post
color: rgb(250, 50, 50)
feature-img: "assets/img/feature-img/network.jpg"
thumbnail: "assets/img/thumbnails/feature-img/network.jpg"
title: "[네트워크 기초]Port와 Switch"
categories: Network
tag: [Network, port, swtich]
toc: true
author_profile: false
sidebar:
 nav: "docs"
#search: false

---

# Port와 Switch 이야기

## Port 번호에 대해

![image](https://user-images.githubusercontent.com/75375944/185777044-0aaac0ff-b69b-4c03-ab0d-41be9cdc5b6d.png)

앞선 포스팅에서 Port 번호의 식별자는 여러가지로 해석된다고 말한 바 있다.

개발자의 관점에서는 Process 얘기를 많이하게 되는데, 본질이 file이지만 프로토콜을 추상화했기때문에 Socket이라 불렀던 이 Socket에 접근하는게 바로 Port 번호이다.

기본적으로 TCP/IP Port 번호는 16bit이다. 그럼 2의 16제곱이니까 0에서 65535까지의 Port 개수가 나온다. (주의할점은 0하고 65535는 쓰면 안되기 때문에 1~65534로 봐야한다.)

여기서 중요한 것은 어떤 Host가 한대있고 IP주소가 192.168.0.10라 가정하고 엣지와 크롬 두개의 프로그램이 실행되었다고하면 각자 소켓을 열 것이다. 그럼 네이버에 접속한다고하면 엣지의 경우 30000번이라고 하면 크롬은 30000번이 아닌 다른 번호를 쓰게되어있다.

앞으로 Packet이라는 데이터 단위를 배우게되면 Packet이라는 단위가 네트워크에 들어오게되면 Host → NIC → Device driver → TCP/IP → Process 순으로 전달이 된다. 여기서 중요한것은 운영체제 수준의 4계층에서 process로 갈것인지 엣지로 갈것인지 크롬으로 갈것인지를 정하는 것이 Port번호이다.

## Switch가 하는일

Network라는 것은 비유적으로 **고속도로**와 비슷하다.

앞서 Network 은 Switch로 이루어져있다고 했다. 그리고 Switch가 하는일이 Swtiching이다. 즉 , 경로선택이자 인터페이스이다.

어떤 출발지에서 자동차 한대가 목적지에 가기위해서 출발하면 어떤 교차로를 만나게되면 어디로갈지 선택을 해야한다. 만약 네비나 지도가 없다고한다면 뭘보고 목적지를 향해가는가? 위의 이정표를 보고 선택해야한다. 이 이정표는 경로선택의 근거이다. 만약에 어떤 경로를 선택했을 때 조금 비효율적인 선택을 했다고 가정하면 어쨋거나 목적지에 도착하게된다.

이때 Switch라고 하는것이 교차로인 것이고 선택하는 것이 Switching이 된다.

어떤 길을 선택하면 목적지에 가지못하거나 돌아가야하는 경우도 생긴다. 따라서 효율적으로 목적지에 도착하는 것이 가장 중요한 것이다. 이것이 네트워킹의 핵심이다.

Internet이라는 것은 L3 Switching하는 Router의 집합체이다. 따라서 교차로는 Router가 된다. Router를 기준으로 여러개의 네트워크 인터페이스가 연결되어있다. 이후, 여러개의 정보가 도착하면 고르는것이 Switching이다.

그때 가장 효율적인 길로 가려면 어떻게 해야하냐는 것을 Router 끼리 프로토콜을 가지고 통신을해서 최적화된 경로를 결정한다. 그리고 인터넷의 좋은점은 Router가 몇개 손상되었다해서 네트워크 전체가 망가지진 않는다.

그리고 이 비유에서 자동차는 Packet이 된다. Packet 단위의 데이터가 교차로, Router에 도착하면 경로선택 Switching을 하게된다. 그중에서 최적화된 경로를 통해서 목적지에 가게 되는 근거가 이정표이다. 이 이정표를 Routing table이라고 한다. 즉 Routing table을 기준으로 의사결정을 하는 것임을 기억하자.
