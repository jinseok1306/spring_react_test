# React + Spring 연동 페이지 구축

### 1. Spring Boot로 프로젝트 생성하기  
- Spring Boot 사이트(https://start.spring.io/) 접속 후 프로젝트 생성 (이때 Dependency는 Spring Web 추가할 것)
- 생성된 프로젝트를 개인 IDE에서 실행 (java 경로에 생성된 프로젝트명.java 파일에서 Run(F5)으로 실행)
- Controller 추가 후 프로젝트 실행하기 (초기 Port : 8080) 테스트는 localhost:8080/hello 로 호출하면 json 데이터를 Return 받을 수 있다. 
<img src="./scan/spring initializr.png"  width="800" >  

### 2. React 프로젝트 생성하기 
- `npx create-react-app 프로젝트명` 프로젝트 생성 
- `npm start`으로 프로젝트 실행
<img src="./scan/react page.png"  width="800" >  

### 3. Proxy Server 설정하기  
로컬 환경에서 개발할 경우 React(Front-End)와 Spring(Back-End)을 각각 실행 후 통신해야된다. 그런데 두 프로젝트의 url이 다르기 떄문에 React 쪽에서 통신할 kv때 CORS문제가 발생해 통신할 수 없다. 그렇기에 개발 시에는 Proxy Server를 설정하여 CORS를 방지하는 것이다.   <a href="https://create-react-app.dev/docs/proxying-api-requests-in-development/">공식문서</a>에 Proxy Server를 설정하는 방법을 참고하여 아래와 같이 설정한다.  

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

먼저 Spring 프로젝트 폴더에 있는 build.gradle 파일에 아래 소스를 추가한다. 
```gradle
//React Project를 빌드 시 추가하기위해 설정
def frontendDir = "$projectDir/src/main/client_app"

sourceSets {
    main {
        resources {
            srcDirs = ["$projectDir/src/main/resources"]
        }
    }
}

task installReact(type: Exec) {
    workingDir "$frontendDir"
    inputs.dir "$frontendDir"
    group = BasePlugin.BUILD_GROUP
    if (System.getProperty('os.name').toLowerCase(Locale.ROOT).contains('windows')) {
        commandLine "npm.cmd", "audit", "fix"
        commandLine 'npm.cmd', 'install'
    } else {
        commandLine "npm", "audit", "fix"
        commandLine 'npm', 'install'
    }
}

task buildReact(type: Exec) {
    dependsOn "installReact"
    workingDir "$frontendDir"
    inputs.dir "$frontendDir"
    group = BasePlugin.BUILD_GROUP
    if (System.getProperty('os.name').toLowerCase(Locale.ROOT).contains('windows')) {
        commandLine "npm.cmd", "run-script", "build"
    } else {
        commandLine "npm", "run-script", "build"
    }
}

task copyReactBuildFiles(type: Copy) {
    dependsOn "buildReact"
    from "$frontendDir/build"
    into "$buildDir/resources/main/static"
}

//배포시에만 빌드파일 포함
tasks.bootJar {
    dependsOn "copyReactBuildFiles"
}
```  
위 내용을 간단히 정리하면 Spring Boot 프로젝트가 build될 때 React 프로젝트를 먼저 build하고 결과물을 Spring Boot 프로젝트 build 결과물에 포함시킨다는 스크립트이다. 처리 순서는 processResources를 기점으로 installReact->buildReact->copyReactBuildFiles 순으로 실행된다. 

이제 build.gradle 파일 저장 후 프로젝트 경로에서 `./gradlew build` 명령어를 실행하여 Build를 진행한다. 만약 이때 `ERR_OSSL_EVP_UNSUPPORTED` 오류가 발생할 경우 React 프로젝트에 package.json 파일에 react-scripts 버전이 최신버전이 아니여서 Open SSL3 버전 규격에 어긋나 발생하는 오류이다. react-scrips 버전을 최신버전으로 변경 후 설치하면 오류가 해결된다.  

### 5. 서버에 배포하기  
Build된 파일은 프로젝트 build/lib 경로에 있으며 .jar 확장자로 저장된다. 프로젝트를 실행하려면 cli 환경에서 해당 경로로 이동 후 `java -jar [name-of-jar-file].jar`로 입력하면 프로젝트를 실행할 수 있고 localhost:8080으로 접속하면 React 화면이 나온다. 

실제 서버에 배포할 때는 해당 서버에 JDK를 설치하고 jar을 실행한 다음에 백그라운드로 jar을 `nohup java –jar [name-of-jar-file].jar &
` 명령어로 배포하면 된다. 이후 Tomcat으로 컴파일 경로를 설정하고 DNS 서버에서 IP와 호스트명을 일치시키면 웹 퍼블리싱 작업이 완료된다.  


출처
- https://7942yongdae.tistory.com/136  
- https://velog.io/@kcdoggo/%EC%8A%A4%ED%94%84%EB%A7%81%EB%A6%AC%EC%95%A1%ED%8A%B8-%EC%97%B0%EB%8F%99 
- https://trace90.tistory.com/entry/SpringBoot-React-%ED%80%B5%EC%8A%A4%ED%83%80%ED%8A%B8-Gradle
- https://deeplify.dev/back-end/spring/executable-jar