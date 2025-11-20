---
name: 고진무 (Jinmu Go)
philosophy: |
  The 撫 (mu) part of my name means "to touch the formless and make it sink".
  I want to live by discovering and solving problems that are not visible on the surface.
contact:
  email: hi@jinmu.me
  github: https://github.com/JinmuGo
hero:
  tagline: SOLUTIONIST
  title: Jinmu Go
  description: |-
    A Smooth Sea Never
    Made a Skilled Sailor
about:
  bio:
    - A YouTube video I stumbled upon changed the course of my life. After realizing that I could communicate with the world through programming, I decided to become a programmer.
    - The name 撫 (nothing to touch) is made by combining the characters 手 (hand) and 無 (nothing), which means to touch the intangible things (heart, soul) with your hands to make them go away.
    - Just like the name, I want to live my life by touching and soothing people's hearts.
  caption: 撫 (mu) - touching and soothing the shapeless things
sections:
  - type: experiences
    title: Experiences and activities
    data:
      - id: 1
        name: AIMuse
        tagline: Transforming the exhibition experience with AI docent services
        role: Frontend, DevOps
        award: Excellence Award
        description: |
          An interactive docent service utilizing AI technology. Visitors can get in-depth information by talking to a chatbot in front of the exhibits. We connected the React-based frontend with the AI backend and built a CI/CD pipeline for the entire system.
        problem: When visiting an exhibition, it was difficult to hear the docent's explanation or get in-depth information about the artwork you were curious about.
        responsibilities:
          - Wireframe design and UI/UX design.
          - Implemented the exhibition-artwork description page and AI chatbot interface
          - Build a front-end to back-end CI/CD pipeline and automate deployment
        image: /project/aimuse.png
        category: AI & Web Development
        tags:
          - React
          - AI
          - CI/CD
          - Docker
          - DevOps
        timeline: Jan 2025
        github: https://github.com/2025-IA-x-AI-Hackathon/Hack-gotowork
        exhibitNumber: I
        year: 2025
        metrics:
          users: Award of Excellence
          performance: AI integration
          impact: Innovative UX
      - id: 2
        name: Hey, Grandma
        tagline: A platform connecting Jeju parents and mainland children
        role: Frontend Developer
        description: A web platform that connects parents on Jeju Island with their children living on the mainland. I collaborated with planners and designers to implement a user-centered interface and completed the actual deployment. I have experience in modern web development utilizing React and TypeScript.
        problem: We wanted to solve the problem of difficult communication and connection between Jeju Island parents and their mainland children.
        responsibilities:
          - Collaborated with planners and designers to implement user-centered UI/UX.
          - React-based front-end development and state management
          - Production environment deployment and performance optimization
        image: /project/hey-oldlady.png
        category: Web Development
        tags:
          - React
          - TypeScript
          - Frontend
          - Collaboration
        timeline: "2024"
        github: https://github.com/ddol-mang/hey-oldlady
        exhibitNumber: II
        year: 2024
        metrics:
          users: 12 cloudtons
          performance: Collaborative project
          impact: Real-world deployments
  - type: opensource
    title: Open source
    data:
      - name: obsidian-theme-by-folder
        description: |
         An Obsidian plugin that automatically switches the Obsidian theme based on the folder of the open note.
         Developed because I wanted to apply a different Obsidian theme than my usual one only when writing.
        role: Creator
        tech: TypeScript, Obsidian API
        github: https://github.com/JinmuGo/obsidian-theme-by-folder
        githubRepo: JinmuGo/obsidian-theme-by-folder
        obsidianPluginId: theme-by-folder
      - name: obsidian-go-up
        description: |
          Obsidian plugin to quickly navigate to a specified 'parent' page
          Developed to quickly navigate to logically connected parent documents
        role: Creator
        tech: TypeScript, Obsidian API
        github: https://github.com/JinmuGo/obsidian-go-up
        githubRepo: JinmuGo/obsidian-go-up
        obsidianPluginId: go-up
      - name: sls
        description: |
          Interactive SSH connection assistant CLI tool developed in Go
          Developed so that you can often forget the hostname of the SSH config or get confused about which hosts are there, so you can see the ssh config at a glance like ls and quickly select and connect with fzf
        role: Creator
        tech: Go
        github: https://github.com/JinmuGo/sls
        githubRepo: JinmuGo/sls
  - type: skills
    title: Technology stack
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
            level: Advanced
          - name: GitHub Actions
            level: Advanced
          - name: AWS Beanstalk
            level: Intermediate
          - name: AWS ElastiCache
            level: Intermediate
          - name: AWS RDS
            level: Intermediate
          - name: Coolify
            level: Intermediate
  - type: certifications
    title: Certifications
    data:
      - name: AWS CLF-C02
        link: https://jinmu.me/posts/aws-clf-c02
  - type: projects
    title: Projects
    data:
      - id: 5
        name: FdF
        title: FdF
        tagline: 3D Height Map Wireframe Renderer.
        role: Graphics Programmer
        description: |
          This is a graphics project that reads 3D height map data and renders it into 2D wireframes. I used C language and OpenGL (MiniLibX) to implement basic principles of computer graphics such as 3D coordinate transformations, projections, and rotations.
        problem: We needed to effectively represent points in 3D space on a 2D screen and implement real-time interactivity.
        responsibilities:
          - Implemented 3D → 2D projection and isometric algorithms.
          - Optimize line drawing using the Bresenham algorithm
          - Implemented rotation, zooming, and panning via keyboard input
        image: /project/fdf.png
        category: Graphics Programming
        tags:
          - C
          - OpenGL
          - 3D Graphics
          - Computer Graphics
        timeline: Dec 2022 - Jan 2023
        github: https://github.com/JinmuGo/fdf
        exhibitNumber: V
        year: 2023
        metrics:
          users: 3D rendering
          performance: Real-time
          impact: Understanding graphics
      - id: 4
        name: Minishell
        title: Minishell
        tagline: A Bash-like shell implemented in C
        role: System Programmer
        description: |
          Implemented a Unix shell from scratch using C language and system calls. Implemented core shell features such as pipes, redirection, and signal processing, and optimized performance by applying my own hash tables for managing environment variables.
        problem: I needed to understand the inner workings of the shell, and implement efficient environment variable management and command processing.
        responsibilities:
          - Implemented core shell features such as pipes, redirection, signal handling, etc.
          - Optimized environment variable management by implementing hash tables directly
          - Designed lexers, parsers, and developed the command execution engine
        image: /project/minishell.png
        category: System Programming
        tags:
          - C
          - System Programming
          - Shell
          - Hash Table
          - Unix
        timeline: Jan 2023 - Mar 2023
        github: https://github.com/JinmuGo/minishell
        exhibitNumber: IV
        year: 2023
        metrics:
          users: Shell implementation
          performance: Hash Tables
          impact: System understanding
      - id: 3
        name: Webserv
        title: Webserv
        tagline: A high-performance static web server implemented in C++.
        role: Backend Developer, System Architect
        description: |
          This is a project that implemented an HTTP web server in C++ from scratch. Designed an asynchronous I/O multiplexing structure using kqueue, led an object-oriented refactoring based on the Single Responsibility Principle (SRP), and gained a deep understanding of systems programming and network protocols.
        problem: Gained a deep understanding of the HTTP protocol and the inner workings of a web server, and needed to implement efficient asynchronous processing.
        responsibilities:
          - Designed and implemented a kqueue-based asynchronous event loop.
          - Developed HTTP/1.1 protocol parsing and response processing logic
          - Led object-oriented refactorings based on the Single Responsibility Principle (SRP)
        image: /project/webserv.png
        category: System Programming
        tags:
          - C
          - Web Server
          - kqueue
          - I/O Multiplexing
          - OOP
        timeline: Oct 2023 - Dec 2023
        github: https://github.com/WebWaveMaker/webserv
        exhibitNumber: III
        year: 2023
        metrics:
          users: High Performance
          performance: Asynchronous I/O
          impact: architectural design
---

