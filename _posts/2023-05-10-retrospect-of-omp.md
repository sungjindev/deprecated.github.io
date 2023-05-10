---
title: 일주일 간의 어버이날 관련 웹 애플리케이션 해커톤 회고록 
categories: [Computer engineering, Cloud infrastructure]
tags: [infrastructure, dockerizing, spring, load balancer, spring security, JWT, https, CI/CD, 인프라, 도커라이징, 스프링, 로드 밸런서, 스프링 시큐리티]
---

이번 포스팅에서는 어버이날을 기념해 일주일 간 진행한 어버이날 관련 웹 애플리케이션 해커톤에 대한 회고 및 맞닥뜨렸던 중요한 이슈들에 대해 이야기해 보려고 합니다. 크게 Spring Security를 활용한 JWT 인증 문제, https 배포를 위한 로드 밸런서 추가 문제 등이 있었는데 관련된 문제에 직면해있는 분들에게 도움이 될 것이라고 생각합니다.

## Oh My Parents! 프로젝트 소개
우선 일주일동안 어떤 프로젝트를 어떻게 진행했는지 소개해보려고 합니다. 저는 Backend Engineer 2명, Frontend Engineer 2명, Designer 1명으로 이루어진 팀에서 Backend Engieer로서 회원과 관련된 비즈니스 로직, 인증 및 소셜 로그인 기능 등을 개발하고 CI/CD 구축 및 클라우드 인프라 배포 총괄을 맡았습니다.
따라서 제가 해커톤을 진행하면서 맞닥뜨렸던 문제들은 모두 이와 관련되어 크게 Spring Security를 활용한 JWT 인증 문제, https 배포를 위한 로드 밸런싱 문제 등이 있었고 이번 포스팅에서는 이러한 이슈들에 대해 자세히 적어보려고 합니다.   
그 전에 앞서, [Oh My Parents!](https://my-parents.day/) 프로젝트는 웹 애플리케이션 서비스로서 지금도 운영되고 있으며 서비스 정식 배포를 시작한지 하루만에 초기 가입 유저 약 150명이라는 저에게 굉장히 의미있는 결과를 가져다 준 프로젝트입니다.

![0](/assets/img/retrospect_of_omp/0.png){: w="80%" h="80%" style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

![1](/assets/img/retrospect_of_omp/1.png){: w="80%" h="80%" style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

![2](/assets/img/retrospect_of_omp/2.png){: w="80%" h="80%" style="border:1px solid #eaeaea; border-radius: 7px; padding: 0px;"}

위 화면에서 보시는 것처럼 어머니, 아버지에 대한 질문 10개가 설정되어 있고 선택하여 질문에 대한 답을 적고 이를 부모님에게 링크로 공유하여 직접 채점을 받아 부모님에 대해 얼마나 알고 있는지 알아볼 수 있는 간단한 서비스입니다.   
이는 바쁜 현대 사회를 살면서 무조건적인 부모님의 사랑 아래 우리는 과연 부모님에 대해 얼마나 알고 있는지 정확히 깨닫고 더욱 좋은 관계를 이어나가기 위한 목적으로 기획되었습니다.


## Spring Security를 활용한 JWT 인증 이슈
카카오 소셜 로그인을 잘 연동한 뒤에, 카카오 프로필을 통해 얻어온 고객의 정보를 바탕으로 내부 회원 가입을 진행하고 Spring Security를 사용해 JWT 토큰을 발행하고자 하였습니다. 이때 계속해서 NullPointerException이 발생했었는데 이 문제가 나중에 보니 굉장히 간단한 문제였지만 제법 많은 시간을 들여 해결할 수 있었습니다.
바로 그 원인은 Spring Security에서 Role을 바탕으로 사용자 권한 설계를 하게 되는데 이때 Role을 Array 형태로 저장했지만 빈 배열이 아니고 NULL 값을 가지도록 구현된 부분이 있었어서 발생했던 에러입니다. 배열을 활용한다면 빈 배열과 NULL을 명확히 구분하여 이러한 실수를 하지 않도록 주의해야겠습니다.

## https 이슈
프론트엔드와 RESTful API로 


## References
> https://minkukjo.github.io/devops/2020/08/28/Infra-22/   

