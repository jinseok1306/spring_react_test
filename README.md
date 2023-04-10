# React + Spring 연동 페이지 구축

### 1. Spring Boot로 프로젝트 생성하기  
- Spring Boot 사이트(https://start.spring.io/) 접속 후 프로젝트 생성 (이때 Dependency는 Spring Web 추가할 것)
- 생성된 프로젝트를 개인 IDE에서 실행 (java 경로에 생성된 프로젝트명.java 파일에서 Run(F5)으로 실행)
- Controller 추가 후 프로젝트 실행하기 (초기 Port : 8080) 테스트는 localhost:8080/hello 로 호출하면 json 데이터를 Return 받을 수 있다. 
<img src="./scan/spring initializr.png"  width="800" >  

### 2. React 프로젝트 생성하기 
- `npx create-react-app 프로젝트명` 프로젝트 생성 
- `npm run`으로 프로젝트 실행
<img src="./scan/react page.png"  width="800" >  

### 3. Proxy Server 설정하기  
로컬 환경에서 개발할 경우 React(Front-End)와 Spring(Back-End)을 각각 실행 후 통신해야된다. 그런데 두 프로젝트의 url이 다르기 떄문에 React 쪽에서 통신할 때 CORS문제가 발생해 통신할 수 없다. 그렇기에 개발 시에는 Proxy Server를 설정하여 CORS를 방지하는 것이다.   <a href="https://create-react-app.dev/docs/proxying-api-requests-in-development/">공식문서</a>에 Proxy Server를 설정하는 방법을 참고하여 아래와 같이 설정한다.  

- `npm install http-proxy-middleware --save ` 명령어로 라이브러리 설치
- src 경로에 setupProxy.js 파일 생성 후 아래와 같이 소스 작성  
```javascript
const { createProxyMiddleware } = require('http-proxy-middleware');

module.exports = function(app) {
  app.use(
    '/api', //개발 환경에서 해당 경로로 시작하는 url 호출 시 proxy Server를 사용
    createProxyMiddleware({
      target: 'http://localhost:8080',
      changeOrigin: true,
    })
  );
};
```

Proxy Server설정을 완료했다면 이제 React에서 아래와 같이 경로 설정 후 호출하면 개발 환경에서는 Proxy Server로 호출하고 빌드 후 배포한 운영 환경에서는 동일한 주소이기에 Path로 바로 접근한다.
```javascript
useEffect(()=>{
    fetch("/api/hello")
      .then((response) => {
        console.log(response.json())
        return;
      })
      .then(function (data) {
        setMessage(data);
      });
  }, []);

```  

### 4. Gradle을 이용해서 프로젝트 빌드하기  
CI/CD 환경을 구축하기 위해서는 프로젝트가 잘 패키지화 될 수 있도록 해주어야한다. 





출처
- https://7942yongdae.tistory.com/136  
- https://velog.io/@kcdoggo/%EC%8A%A4%ED%94%84%EB%A7%81%EB%A6%AC%EC%95%A1%ED%8A%B8-%EC%97%B0%EB%8F%99