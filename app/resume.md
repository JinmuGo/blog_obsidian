---
name: 고진무 (Jinmu Go)
philosophy: |-
  이름 중 撫(무)는 '형체 없는 것을 손으로 어루만져 가라앉힌다'는 뜻을 담고 있습니다.
  겉으로 드러나지 않는 문제를 발견하고 해결하며 살아가고자 합니다.
contact:
  email: hi@jinmu.me
  github: https://github.com/JinmuGo
sections:
  - type: experiences
    title: 경험 및 활동
    data:
      - id: 1
        name: IA x AI 해커톤
        tagline: AI 도슨트 서비스 개발
        description: |- 
          AI 도슨트 서비스 'AIMuse' 기획 및 개발.
          와이어프레임 디자인, 전시·작품 설명 페이지와 챗봇구현, 프론트엔드·백엔드 CI/CD 구축
        problem: 미술관 관람객들이 작품에 대한 깊이 있는 정보를 얻기 어려운 문제 해결
        role: Frontend, DevOps
        responsibilities:
          - 와이어프레임 디자인
          - 전시·작품 설명 페이지 구현
          - 챗봇 인터페이스 개발
          - CI/CD 파이프라인 구축
        image: /placeholder.png
        category: Hackathon
        tags:
          - Frontend
          - DevOps
          - CI/CD
        timeline: "2025"
        github: https://github.com/2025-IA-x-AI-Hackathon/Hack-gotowork
        live: https://www.epnc.co.kr/news/articleView.html?idxno=324404
        exhibitNumber: "01"
        year: 2025
        award: 우수상
      - id: 2
        name: 구름톤(Goormthon) 12기
        tagline: 제주 부모-육지 자식 연결 플랫폼
        description: 기획자·디자이너와 협업하여 제주 부모-육지 자식 간 연결 플랫폼 '어이, 할망' 개발 및 배포
        problem: 제주도에 거주하는 부모와 육지에 있는 자식 간 소통 문제 해결
        role: Frontend Developer
        responsibilities:
          - 프론트엔드 개발
          - 기획자·디자이너와 협업
          - 서비스 배포
        image: /placeholder.png
        category: Bootcamp
        tags:
          - Frontend
          - React
          - Collaboration
        timeline: "2024"
        github: https://github.com/ddol-mang/hey-oldlady
        live: https://jinmu.me/posts/12th-9oormthon
        exhibitNumber: "02"
        year: 2024
  - type: opensource
    title: 오픈소스
    data:
      - name: obsidian-theme-by-folder
        description: |-
          열린 노트의 폴더에 따라 Obsidian 테마를 자동으로 전환되는 옵시디언 플러그인.
          평소쓰던 옵시디언 테마와 다른 테마를 글을 쓸 때만 적용하고 싶어서 개발
        role: Creator
        tech: TypeScript, Obsidian API
        github: https://github.com/JinmuGo/obsidian-theme-by-folder
        githubRepo: JinmuGo/obsidian-theme-by-folder
        obsidianPluginId: theme-by-folder
      - name: obsidian-go-up
        description: |-
          지정된 '상위' 페이지로 빠르게 이동하는 옵시디언 플러그인.
          논리적으로 연결된 상위 문서를 빠르게 이동하고 싶어서 개발
        role: Creator
        tech: TypeScript, Obsidian API
        github: https://github.com/JinmuGo/obsidian-go-up
        githubRepo: JinmuGo/obsidian-go-up
        obsidianPluginId: go-up
      - name: sls
        description: |-
          Go로 개발한 대화형 SSH 연결 도우미 CLI 도구.
          SSH config의 호스트 이름을 자주 잊거나 어떤 호스트가 있는지 헷갈려서, ssh config를 ls처럼 한눈에 보고 fzf로 빠르게 선택해 연결할 수 있도록 개발
        role: Creator
        tech: Go
        github: https://github.com/JinmuGo/sls
        githubRepo: JinmuGo/sls
  - type: skills
    title: 기술 스택
    data:
      - category: Backend
        items:
          - name: Node.js/TypeScript
            level: Advanced
          - name: C/C++
            level: Advanced
          - name: Go
            level: Intermediate
      - category: Frontend
        items:
          - name: React
            level: Advanced
          - name: TypeScript
            level: Advanced
          - name: Astro
            level: Intermediate
          - name: Next.js
            level: Intermediate
      - category: DevOps & Cloud
        items:
          - name: Docker
            level: Intermediate
          - name: GitHub Actions
            level: Intermediate
          - name: AWS Beanstalk
            level: Beginner
          - name: AWS ElastieCache
            level: Beginner
          - name: AWS RDS
            level: Beginner
          - name: Coolify
            level: Beginner
  - type: certifications
    title: 자격증
    data:
      - name: AWS CLF-C02
        link: https://jinmu.me/posts/aws-clf-c02
  - type: projects
    title: 프로젝트
    data:
      - id: 3
        name: FdF
        title: FdF
        tagline: 3D 와이어프레임 렌더링
        description: 3D 높이 맵을 읽어 2D 와이어프레임으로 렌더링하는 그래픽 프로젝트.
        problem: 3D 지형 데이터를 2D 화면에 효과적으로 시각화
        role: Developer
        responsibilities:
          - OpenGL을 이용한 렌더링 구현
          - 3D to 2D 변환 알고리즘 구현
        image: /project/fdf.png
        category: Graphics
        tags:
          - C
          - OpenGL
          - 3D Graphics
        timeline: "2022.12 ~ 2023.01"
        github: https://github.com/JinmuGo/fdf
        live: https://github.com/JinmuGo/fdf
        exhibitNumber: "03"
        year: 2022
      - id: 4
        name: Minishell
        title: Minishell
        tagline: Bash 유사 셸 구현
        description: C로 구현한 Bash 유사 셸. C 언어와 시스템 콜을 사용하여 유닉스 셸을 구현, 환경 변수 관리를 위해 직접 구현한 해시 테이블을 적용하여 성능 최적화
        problem: 유닉스 셸의 동작 원리 이해 및 구현
        role: Developer
        responsibilities:
          - 셸 파싱 및 실행 로직 구현
          - 해시 테이블 자료구조 설계 및 구현
          - 환경 변수 관리 시스템 개발
        category: System Programming
        tags:
          - C
          - System Programming
          - Shell
          - Hash Table
        timeline: "2023.01 ~ 2023.03"
        github: https://github.com/JinmuGo/minishell
        live: https://github.com/JinmuGo/minishell
        exhibitNumber: "04"
        year: 2023
      - id: 5
        name: Webserv
        title: Webserv
        tagline: C++ 정적 웹 서버
        description: C++로 구현한 정적 웹 서버. kqueue(I/O 멀티플렉싱)를 사용한 비동기 이벤트 루프를 구현, 단일 책임 원칙(SRP) 기반 객체 지향 리팩토링을 주도하여 구조를 개선
        problem: 고성능 비동기 웹 서버 구현
        role: Developer
        responsibilities:
          - kqueue 기반 이벤트 루프 구현
          - HTTP 프로토콜 파싱 및 처리
          - 객체 지향 리팩토링 주도
        category: Web Server
        tags:
          - C++
          - Web Server
          - kqueue
          - I/O Multiplexing
          - OOP
        timeline: "2023.10 ~ 2023.12"
        github: https://github.com/WebWaveMaker/webserv
        live: https://github.com/WebWaveMaker/webserv
        exhibitNumber: "05"
        year: 2023
---

