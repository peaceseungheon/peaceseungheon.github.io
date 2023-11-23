---
  layout: single
  title: 'Spring Security 관련 정리'
  categories:
    - spring
  toc: true
  toc_sticky: true
  tags:
    - SPRING SECURITY
---

사내에서 쓰고 있던 스프링부트 버전은 2.0.4였다. 보안문제도 있고 스프링부트 3점대가 나오고 있는 상황에서 너무 낮은 버전을 쓰고 있는 것 같아 2.7.14로 버전을 업그레이드 했다. 버전이 바뀌면서 스프링 자체적으로 작동방식이 많이 변경되어 여러 수정작업을 거쳤다. 

그 중 Spring Security와 관련한 수정작업을 하며 Spring Security에 대한 이해가 부족함을 느꼈다. 이번 기회에 Spring Security를 공부하겠다는 마음으로 글을 작성해봤다.

## Spring Security에 대해 알아보자