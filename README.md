# React + Spring 연동 페이지 구축

### 1. Spring Boot로 프로젝트 생성하기  
- Spring Boot 사이트(https://start.spring.io/) 접속 후 프로젝트 생성 (이때 Dependency는 Spring Web 추가할 것)
- 생성된 프로젝트를 개인 IDE에서 실행
- Controller 추가 후 프로젝트 실행하기 (초기 Port : 8080) 테스트는 localhost:8080/hello 로 호출하면 json 데이터를 Return 받을 수 있다.  
<img src="./scan/spring initializr.png"  width="400" >  

### 2. React 프로젝트 생성하기 
```cmd
npx create-react-app 프로젝트명
``` 


출처 : https://7942yongdae.tistory.com/136