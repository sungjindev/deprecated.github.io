---
title: 일주일 간의 어버이날 관련 웹 애플리케이션 해커톤 회고록 
categories: [Computer engineering, Cloud infrastructure]
tags: [infrastructure, dockerizing, spring, load balancer, spring security, JWT, https, CI/CD, 인프라, 도커라이징, 스프링, 로드 밸런서, 스프링 시큐리티]
---

이번 포스팅에서는 어버이날을 기념해 일주일 간 진행한 어버이날 관련 웹 애플리케이션 해커톤에 대한 회고 및 맞닥뜨렸던 중요한 이슈들에 대해 이야기해 보려고 합니다. 크게 Spring Security를 활용한 JWT 인증 문제, https 배포를 위한 로드 밸런서 추가 문제 등이 있었는데 관련된 문제에 직면해있는 분들에게 도움이 될 것이라고 생각합니다.

## Oh My Parents! 프로젝트 소개
우선 일주일동안 어떤 프로젝트를 어떻게 진행했는지 소개해보려고 합니다. 저는 Backend Engineer 2명, Frontend Engineer 2명, Designer 1명으로 이루어진 팀에서 Backend Engieer로서 회원과 관련된 비즈니스 로직, 인증 및 소셜 로그인 기능 등을 개발하고 CI/CD 구축 및 클라우드 인프라 배포 총괄을 맡았습니다.
따라서 제가 해커톤을 진행하면서 맞닥뜨렸던 문제들은 모두 이와 관련되어 크게 Spring Security를 활용한 JWT 인증 문제, https 배포를 위한 로드 밸런싱 문제 등이 있었고 이번 포스팅에서는 이러한 이슈들에 대해 자세히 적어보려고 합니다.   
그 전에 앞서, [Oh My Parents!](https://my-parents.day/) 프로젝트는 웹 애플리케이션 서비스로서 지금도 운영되고 있으며 서비스 정식 배포를 시작한지 하루만에 초기 가입 유저 약 150명, 하루동안 총 이용자 약 600명이라는 저에게 굉장히 의미있는 결과를 가져다 준 프로젝트입니다.

![0](/assets/img/retrospect_of_omp/0.png){: w="80%" h="80%" style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

![1](/assets/img/retrospect_of_omp/1.png){: w="80%" h="80%" style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

![2](/assets/img/retrospect_of_omp/2.png){: w="80%" h="80%" style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

위 화면에서 보시는 것처럼 어머니, 아버지에 대한 질문 10개가 설정되어 있고 선택하여 질문에 대한 답을 적고 이를 부모님에게 링크로 공유하여 직접 채점을 받아 부모님에 대해 얼마나 알고 있는지 알아볼 수 있는 간단한 서비스입니다.   
이는 바쁜 현대 사회를 살면서 무조건적인 부모님의 사랑 아래 우리는 과연 부모님에 대해 얼마나 알고 있는지 정확히 깨닫고 더욱 좋은 관계를 이어나가기 위한 목적으로 기획되었습니다.


## Spring Security를 활용한 JWT 인증 이슈
카카오 소셜 로그인을 잘 연동한 뒤에, 카카오 프로필을 통해 얻어온 고객의 정보를 바탕으로 내부 회원 가입을 진행하고 Spring Security를 사용해 JWT 토큰을 발행하고자 하였습니다. 이때 계속해서 NullPointerException이 발생했었는데 이 문제가 나중에 보니 굉장히 간단한 문제였지만 제법 많은 시간을 들여 해결할 수 있었습니다.
바로 그 원인은 Spring Security에서 Role을 바탕으로 사용자 권한 설계를 하게 되는데 이때 Role을 Array 형태로 저장했지만 빈 배열이 아니고 NULL 값을 가지도록 구현된 부분이 있었어서 발생했던 에러입니다. 배열을 활용한다면 빈 배열과 NULL을 명확히 구분하여 이러한 실수를 하지 않도록 주의해야겠습니다.

## https 이슈
프론트엔드와 API로 통신하면서 Mixed Content 에러가 발생하였습니다. 이는 https를 사용하고 있는 프론트엔드에서 http를 사용하고 있는 백엔드로 요청을 보낼 때 보안되지 않은 프로토콜로 downgrade를 해야하기 때문에 보안에 취약점이 발생할 수 있어 대부분의 브라우저에서 이를 막는 것입니다. 이를 해결하기 위해서는 각 사용자의 브라우저에서 임시적으로 해당 기능을 해제할 수도 있지만 서비스를 해야하는 입장에서 이는 가능한 선택지가 아니였습니다. 그래서 Backend Application에서도 https를 사용하기로 결정하고 Backend가 올라가 있는 Google Cloud Platform에서 Backend 앞 단에 https를 사용하는 로드 밸런서를 붙여 이를 해결하도록 하였습니다.   
이때 로드 밸런서의 프론트엔드에서 백엔드 서비스로 잘 도달할 수 있도록 포트 매핑을 해줘야하는데 Spring Application 내 Apache tomcat이 default port로 사용하고 있는 8080 포트로의 매핑이 원활히 이루어지지 않아서 로드 밸런서의 프론트엔드 포트인 80 포트로 통일하여 포트 매핑 이슈를 해결하였습니다.
이후 로드 밸런서에 도메인을 붙여줬고 https 사용을 위해 SSL 인증서는 기존 Frontend Application에서 사용하고 있던 도메인과 동일한 도메인을 사용하도록 설정해뒀기 때문에 동일한 SSL 인증서를 사용했습니다. 하지만 이후 Backend Application의 도메인이 너무 쉽게 노출되어 버려 보안 이슈가 생길 것 같아서 api.domain.com 과 같이 sub domain을 생성하였는데 **curl 60 SSL: no alternative certificate subject name matches target host name 에러**가 발생했습니다. 이는 표준 SSL 인증서는 하나의 도메인만을 인증할 수 있음을 의미하며 이를 해결하기 위해서는 sub domain을 위한 별도의 SSL 인증서를 추가 발급받거나 조금 더 비용이 비싼 와일드카드 SSL 인증서를 발급받아야 합니다. 저는 하나의 sub domain만을 사용할 예정이었기 때문에 와일드카드 SSL을 구매하는 것은 오버 스펙이라 생각하여 별도의 SSL 인증서를 추가 발급하였습니다. 이를 통해 모든 이슈를 해결할 수 있었습니다.

## 회고
중간 고사 시험 및 기업 면접이 모두 끝나자마자 갑작스럽게 기획된 일주일 간의 해커톤이었고 보통 중장기적인 프로젝트만 진행하다가 단기 프로젝트를 하려다 보니 처음엔 좀 낯설기도 했습니다. 짧은 기간답게 프론트엔드와 배포 예정일에 임박해서 API를 맞추다보니 https 업그레이드와 같은 이슈가 발생하기도하고 그 과정에서 많은 어려움도 있었습니다. 하지만, 그 누구 한명이라도 자기 파트를 완료하지 못하면 모두의 일주일 간의 노력들이 물거품 될 수 있었기 때문에 끝까지 모두 포기하지 않고 최선을 다했고 그 결과 어버이날 늦은 밤에 정식 배포를 진행할 수 있었습니다. 비록 예정된 시간보다 조금 딜레이되긴 했지만 서비스 첫날 신규 가입자 150명이 되고 사용자 입장에서 보이진 않지만 항상 보안 이슈를 걱정하며 fit하게 설정한 방화벽, 소셜 로그인 및 인증 등 제 노력들이 잘 support해주는 것 같아서 많은 보람을 느꼈습니다. 이번 첫 해커톤을 시작으로 더욱 성장하고 더 유익한 애플리케이션을 만드는데 기여할 수 있도록 노력할 것입니다.


## References
> https://serverfault.com/questions/566426/does-each-subdomain-need-its-own-ssl-certificate  

