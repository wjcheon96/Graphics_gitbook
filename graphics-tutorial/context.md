---
description: context class design
---

# 🎊 Context 클래스 디자인

### Context

GLFW / OpenGL 사용시, shader, program 등의 OpenGL object들을 관리하고 렌더링하는데 쓰이는 객체로 쓴다.

* 프로그램 라이프사이클을 고려하여 코드 리팩토링
  * GLFW / OpenGL Context / GLAD 초기화
  * **그림을 그리기 위한 OpenGL objects 생성 (shader / program)**
  * **렌더링**
  * **OpenGL objects 제거**
  * GLFW 종료 / 프로그램 종료

결과론적으로 이 위에서 쓴 shader, program, render에 대한 내용이 해당 객체 안에 전부 들어가게 되며, 이후에 기술하게 되는 coodernate, camera 등의 처리도 해당 context 객체에서 이루어진다.

