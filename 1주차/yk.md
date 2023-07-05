# 1장 익스프레스 소개

- 클라이언트의 HTTP 요청을 받고 HTTP 요청 반환
- 웹 어플리케이션 프레임워크의 서버 부분
- 서버 사이드 어플리케이션(서버 사이드 렌더링, SSR)
  - 어플리케이션의 패이지를 서버에서 렌더링, 클라이언트에게 전송
- 클라이언트 사이드 어플리케이션(싱글 페이지 어플리케이션, SPA)
  - 사용자 인터페이스를 클라이언트에서 렌더링
- 최소한의 서버를 목표로 개발됨
  - 싱글 스레드 사용
- 독립적인 플랫폼
- 노드 어플리케이션 개발할땐 라이센스에 주의

# 2장 노드 시작하기

- nodemon 설치

  - `npm install -g nodemon`
  - 코드를 수정했을때 자동으로 노드 프로그램을 재시작하는 유틸리티
  - 노드의 철학 : 이벤트 주도 프로그래밍
  - 노드에서는 앱이 곧 웹 서버, 즉 웹 서버ㄹ 만들 수 있는 프레임워크를 제공함
  - http.createServer() 메서드는 매개변수로 함수를 받고 http 요청이 일어날때마다 그 함수를 실행시킴

    ```javascript
    const http = require("http");
    const port = process.env.PORT || 3000;

    const server = http.createServer((req, res) => {
      res.writeHead(200, { "Content-Type": "text/plain" });
      res.end("Hello world!");
    });

    server.listen(port, () =>
      console.log(
        `server started on port ${port}; ` + "press Ctrl-C to terminate...."
      )
    );
    ```

  - 라우팅 : 클라이언트가 요청한 콘텐츠를 전송하는 메커니즘

  ```js
  const http = require("http");
  const port = process.env.PORT || 3000;

  const server = http.createServer((req, res) => {
    // normalize url by removing querystring, optional
    // trailing slash, and making it lowercase
    const path = req.url.replace(/\/?(?:\?.*)?$/, "").toLowerCase();
    switch (path) {
      case "":
        res.writeHead(200, { "Content-Type": "text/plain" });
        res.end("Homepage");
        break;
      case "/about":
        res.writeHead(200, { "Content-Type": "text/plain" });
        res.end("About");
        break;
      default:
        res.writeHead(404, { "Content-Type": "text/plain" });
        res.end("Not Found");
        break;
    }
  });

  server.listen(port, () =>
    console.log(
      `server started on port ${port}; ` + "press Ctrl-C to terminate...."
    )
  );
  ```

  - 정적 자원 전송
  - 노드는 정적파일을 열고 콘텐츠를 읽어서 보내야함

    ```js
    const http = require("http");
    const fs = require("fs");
    const port = process.env.PORT || 3000;

    function serveStaticFile(res, path, contentType, responseCode = 200) {
      fs.readFile(__dirname + path, (err, data) => {
        if (err) {
          res.writeHead(500, { "Content-Type": "text/plain" });
          return res.end("500 - Internal Error");
        }
        res.writeHead(responseCode, { "Content-Type": contentType });
        res.end(data);
      });
    }

    const server = http.createServer((req, res) => {
      // normalize url by removing querystring, optional trailing slash, and
      // making lowercase
      const path = req.url.replace(/\/?(?:\?.*)?$/, "").toLowerCase();
      switch (path) {
        case "":
          serveStaticFile(res, "/public/home.html", "text/html");
          break;
        case "/about":
          serveStaticFile(res, "/public/about.html", "text/html");
          break;
        case "/img/logo.png":
          serveStaticFile(res, "/public/img/logo.png", "image/png");
          break;
        default:
          serveStaticFile(res, "/public/404.html", "text/html", 404);
          break;
      }
    });

    server.listen(port, () =>
      console.log(
        `server started on port ${port}; ` + "press Ctrl-C to terminate...."
      )
    );
    ```

# 3장 익스프레스로 시간 절약하기

- 스캐폴딩 : 프로젝트의 기본 구조를 만드는 것
- 대부분의 프로젝트에는 뼈대가 되는 보일러플레이트가 필요. 하지만 매번 만들 필요없음. 만들어둔 뼈대를 복사하면 됨.
- 익스프레스는 익스프레스 프로젝트를 시작할때 스캐폴딩을 생성하는 유틸리티를 제공함
- 하지만 익스프레스 스캐폴딩은 서버 사이드에서 html을 생성하는 방향에 치중되어 있고 api나 spa에서는 큰 도움이 되지 않음.
- app.get : 라우트 추가 하는 메서드
- 경로 : 라우트, 대소문자, 슬래시, 쿼리스트링은 무시됨
- 함수 : 라우트가 일치할 때 호출할 함수(요청과 응답을 매개변수로 받는대 이때 기본적으로 상태코드는 200을 반환함으로 직접 작성할 필요는 없음.)

  ```js
  const express = require("express");

  const app = express();

  const port = process.env.PORT || 3000;

  // custom 404 page
  app.use((req, res) => {
    res.type("text/plain");
    res.status(404);
    res.send("404 - Not Found");
  });

  // custom 500 page
  app.use((err, req, res, next) => {
    console.error(err.message);
    res.type("text/plain");
    res.status(500);
    res.send("500 - Server Error");
  });

  app.listen(port, () =>
    console.log(
      `Express started on http://localhost:${port}; ` +
        `press Ctrl-C to terminate.`
    )
  );
  ```

  - res.writeHead 와 res.end는 권장사항이 아님.
  - res.send : 응답을 보내는 메서드를 end 대신
  - res.status, res.set : 상태코드를 지정하는 메서드를 writeHead 대신 사용

  - 미들웨어 : 요청과 응답 사이에 위치한 함수
  - 미들웨어는 app.use로 추가함
  - 익스프레스는 미들웨어의 순서가 중요함!
  - 뷰 : 사용자가 보는 것을 책임지는 부분(HTML)
  - 템플릿 프레임워크 : 뷰를 생성하는데 사용하는 도구 (핸들바 추천)
  - 핸들바 사용시 {{{body}}}를 사용하면 html이 그대로 출력됨
  - 뷰 엔진 : 뷰를 생성하는데 사용하는 템플릿 프레임워크를 익스프레스에 통합하는 것
  - 뷰 엔진에서 콘텐츠 타입 text/html과 상태 코드 200을 기본으로 반환하므로 따로 명시할 필요는 없음. 반면 404나 500에 해당하는 폴백 핸들러에는 상태 코드를 정확히 명시해야함.
  - 익스프레스는 미들웨어를 사용해서 정적 파일과 뷰를 처리함.
  - 뷰의 동적 콘텐츠 : 뷰에 데이터를 추가하는 것
