--- 
title: "Event 1"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# Event Report – FCAJ Community Day

### Event Objectives

- Break technological barriers and optimize AI performance.
- Enable Enterprise AI adoption.
- Optimize Cloud infrastructure.
- Inspire and connect the technology community.

### Speakers

- **Anh Tinh** – Build Second Brain
- **Hai Anh** – Friendly AI Assistant with Amazon Quick
- **Thinh** – CloudFront as Your Foundation
- **Team VIB** – 36 Hours with LotusHacks
- **Dao Duc** – Non-Determinism of "Deterministic" LLM Settings
- **Cat Vy** – Enterprise-Grade Multi-Agent System

### Key Highlights

#### LLM Non-Determinism

- Learned that setting **temperature = 0** does not always guarantee identical outputs because of GPU floating-point computation and inference batching.
- Understood that LLMs are probabilistic systems, so applications should be designed to handle output variations instead of expecting perfectly deterministic responses.

#### Enterprise Multi-Agent System

- Explored the **Multi-Agent** architecture, where multiple specialized agents collaborate under the coordination of a Manager Agent.
- Learned about the AWS deployment architecture using **Amazon Bedrock AgentCore, AWS Lambda, Amazon API Gateway, Docker, and Amazon ECR**.

#### Amazon CloudFront

- Learned how **Amazon CloudFront** improves application performance through Edge Locations, HTTP/3, Compression, and Persistent Connections.
- Understood security mechanisms such as **Origin Access Control (OAC)** and **Origin Cloaking** for protecting backend resources.

#### Context Engineering

- Learned that the quality of context is more important than the amount of information provided to an LLM.
- Understood how to build AI assistants using **Retrieval-Augmented Generation (RAG)**, Context Management, and Memory instead of relying only on Prompt Engineering.

### What I Learned

#### AI System Design

- Learned to design AI applications using a **probabilistic mindset**, accepting the non-deterministic nature of LLMs and improving reliability with techniques such as **Majority Voting** and **Structured Output**.
- Shifted my perspective from **Prompt Engineering** to **Context Engineering**, focusing on knowledge storage, retrieval, and memory management.
- Understood that enterprise AI systems must be designed with **security, governance, and scalability** from the very beginning.

#### Cloud Infrastructure

- Learned how to leverage the **AWS Global Backbone** and Amazon CloudFront to improve application performance and reduce latency.
- Understood how to execute lightweight business logic at the edge using **CloudFront Functions** or **Lambda@Edge**.
- Gained knowledge of production-ready AI architectures on AWS with an emphasis on security, scalability, and reliability.

### Applying the Knowledge

#### AI Application Development

- Tune LLM parameters according to specific use cases instead of relying on default settings.
- Implement **Structured Output**, **Majority Voting**, and response validation mechanisms to improve AI reliability.
- Build **Multi-Agent** architectures for complex business workflows.

#### AWS Infrastructure Optimization

- Deploy **Amazon CloudFront** to reduce origin workload, accelerate content delivery, and optimize data transfer costs.
- Apply **Origin Cloaking**, **Origin Access Control (OAC)**, and **AWS WAF** to strengthen application security.
- Use **CloudFront Functions** or **Lambda@Edge** for lightweight processing at edge locations.

#### Enterprise AI Assistant

- Implement **Retrieval-Augmented Generation (RAG)** with a Vector Database to provide accurate contextual information.
- Build a context filtering mechanism to retrieve only relevant information, improving response quality while reducing token usage.
- Utilize AWS AI services such as **Amazon Bedrock**, **Amazon Q**, and AI Agents to automate software development workflows and business processes.

### Event Experience

Participating in the **First Cloud AI Journey (FCAJ) Community Day** was a valuable experience that allowed me to explore the latest trends in Cloud Computing, Artificial Intelligence (AI), and modern software architecture. The event not only provided technical knowledge but also helped me understand how organizations leverage AWS technologies to solve real-world business challenges.

#### Learning from Industry Experts

- AWS experts and technology professionals shared practical experience in designing, deploying, and operating large-scale cloud systems.
- The presentations covered real-world case studies on **Microservices**, **Domain-Driven Design (DDD)**, **Event-Driven Architecture**, **Enterprise AI**, and **Application Modernization**.
- These sessions helped me better understand the **Business-First** approach, where business requirements drive architectural decisions.

#### Technical Experience

- Learned how organizations migrate from **Monolithic Architecture** to **Microservices** to improve scalability, maintainability, and performance.
- Explored **Event Storming**, **Domain-Driven Design**, and **Bounded Contexts** for modeling business processes.
- Gained a deeper understanding of **Publish/Subscribe**, **Point-to-Point**, and **Streaming** integration patterns, as well as the differences between synchronous and asynchronous communication.
- Learned about the evolution of AWS compute services from **Amazon EC2** to **Amazon ECS**, **AWS Fargate**, and **AWS Lambda**, along with the benefits of Serverless Computing.

#### Modern Development Tools

- Explored **Amazon Q Developer**, an AI-powered assistant that supports the Software Development Life Cycle (SDLC), from planning and coding to testing and maintenance.
- Learned how AI-assisted code transformation and AWS Lambda can improve developer productivity and accelerate modernization projects.

#### AI and AWS Solutions

- Learned about **Enterprise AI**, **Multi-Agent Systems**, and **Context Engineering** for building scalable and production-ready AI applications.
- Understood how **Amazon CloudFront** improves application performance while enhancing security through AWS Edge services.

#### Key Takeaways

- System design should always begin with business requirements before selecting technologies.
- Applying **Domain-Driven Design** and **Event-Driven Architecture** enables scalable, maintainable, and resilient systems.
- **Serverless Computing** and AWS AI services improve productivity while reducing operational costs.
- The event provided valuable practical insights into application modernization and strengthened my motivation to continue learning and applying AWS technologies in future projects.

