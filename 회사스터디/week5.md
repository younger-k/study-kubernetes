5주차 - Service & Ingress 
==
---

Service
--

- 파드를 로드밸런싱
- 외부 네임스페이스에서 네트워크 통신을 하려면 파드로 바로 들어가진 않고 서비스를 거침.

### 서비스 타입
- NodePort
  - port range : 30000 ~ 32761
  - 호스트와 쿠버네티스를 연결해줌
  - 거의 `dev`에서만 씀니다
- LoadBalancer


제목2
--