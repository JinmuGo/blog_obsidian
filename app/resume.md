---
name: 고진무 (Jinmu Go)
philosophy: |
  이름 중 撫(무)는 '형체 없는 것을 손으로 어루만져 가라앉힌다'는 뜻을 담고 있습니다.
  겉으로 드러나지 않는 문제를 발견하고 해결하며 살아가고자 합니다.
contact:
  email: hi@jinmu.me
  github: https://github.com/JinmuGo
sections:
  - type: experiences
    title: 경험 및 활동
    data:
      - name: IA x AI 해커톤
        role: Frontend, DevOps
        award: 우수상
        description: |
          AI 도슨트 서비스 'AIMuse' 기획 및 개발. 
          와이어프레임 디자인, 전시·작품 설명 페이지와 챗봇구현, 프론트엔드·백엔드 CI/CD 구축 
        github: https://github.com/2025-IA-x-AI-Hackathon/Hack-gotowork
        news: https://www.epnc.co.kr/news/articleView.html?idxno=324404
      - name: 구름톤(Goormthon) 12기
        role: Frontend Developer
        description: 기획자·디자이너와 협업하여 제주 부모-육지 자식 간 연결 플랫폼 '어이, 할망' 개발 및 배포
        github: https://github.com/ddol-mang/hey-oldlady
        blog: https://jinmu.me/posts/12th-9oormthon
  - type: opensource
    title: 오픈소스
    data:
      - name: obsidian-theme-by-folder
        description: |
         열린 노트의 폴더에 따라 Obsidian 테마를 자동으로 전환되는 옵시디언 플러그인
         평소쓰던 옵시디언 테마와 다른 테마를 글을 쓸 때만 적용하고 싶어서 개발
        role: Creator
        tech: TypeScript, Obsidian API
        github: https://github.com/JinmuGo/obsidian-theme-by-folder
        githubRepo: JinmuGo/obsidian-theme-by-folder
        obsidianPluginId: theme-by-folder
      - name: obsidian-go-up
        description: |
          지정된 '상위' 페이지로 빠르게 이동하는 옵시디언 플러그인
          논리적으로 연결된 상위 문서를 빠르게 이동하고 싶어서 개발
        role: Creator
        tech: TypeScript, Obsidian API
        github: https://github.com/JinmuGo/obsidian-go-up
        githubRepo: JinmuGo/obsidian-go-up
        obsidianPluginId: go-up
      - name: sls
        description: |
          Go로 개발한 대화형 SSH 연결 도우미 CLI 도구
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
          - Node.js/TypeScript
          - C/C++
          - Go
      - category: Frontend
        items:
          - React
          - TypeScript
          - Astro
          - Next.js
      - category: DevOps & Cloud
        items:
          - Docker
          - GitHub Actions
          - AWS Beanstalk
          - AWS ElastieCache
          - AWS RDS
          - Coolify
  - type: certifications
    title: 자격증
    data:
      - name: AWS CLF-C02
        link: https://jinmu.me/posts/aws-clf-c02
  - type: projects
    title: 프로젝트
    data:
      - id: 1
        title: FdF
        description: |
          3D 높이 맵을 읽어 2D 와이어프레임으로 렌더링하는 그래픽 프로젝트.
        tags:
          - C
          - OpenGL
          - 3D Graphics
        period: 2022.12 ~ 2023.01
        link: https://github.com/JinmuGo/fdf
        image: /fdf-preview.png
      - id: 2
        title: Minishell
        description: |
          C로 구현한 Bash 유사 셸. C 언어와 시스템 콜을 사용하여 유닉스 셸을 구현,
          환경 변수 관리를 위해 직접 구현한 해시 테이블을 적용하여 성능 최적화
        tags:
          - C
          - System Programming
          - Shell
          - Hash Table
        period: 2023.01 ~ 2023.03
        link: https://github.com/JinmuGo/minishell
      - id: 3
        title: Webserv
        description: |
          C++로 구현한 정적 웹 서버. kqueue(I/O 멀티플렉싱)를 사용한 비동기 이벤트 루프를 구현,
          단일 책임 원칙(SRP) 기반 객체 지향 리팩토링을 주도하여 구조를 개선
        tags:
          - C++
          - Web Server
          - kqueue
          - I/O Multiplexing
          - OOP
        period: 2023.10 ~ 2023.12
        link: https://github.com/WebWaveMaker/webserv
---

