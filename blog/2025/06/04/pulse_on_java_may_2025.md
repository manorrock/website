# ☕ Pulse on Java – May 2025 Edition  
*Your AI-generated monthly roundup of Java framework and platform updates.*

Welcome to the May 2025 edition of *Pulse on Java*, where we track the latest releases, innovations, and trends across the Java ecosystem. This month brought major GA releases, milestone builds, and AI-native integrations that are reshaping how Java developers build modern applications.

Please feel free to send suggestions for content to info@manorrock.com

## 🔧 Framework Releases & Highlights

### 🌱 Spring AI 1.0.0 GA  
Released on **May 20, 2025**, Spring AI 1.0.0 brings AI-native capabilities to the Spring ecosystem:
- **ChatClient API**: Unified interface for over 20 AI models including OpenAI, Hugging Face, Anthropic, and ZhiPu.
- **Vector store integrations**: Redis, Elasticsearch, MongoDB, Oracle, and more.
- **Tool calling & RAG support**: Built-in support for retrieval-augmented generation and agent workflows.
- **OpenRewrite migration recipe**: Automates upgrades from milestone versions to GA.

### 🧠 LangChain4j 1.0.0 GA  
LangChain4j, the Java-native LLM orchestration library, also hit **1.0.0 GA** in May:
- **Core features**: Prompt chaining, memory, tool calling, and support for OpenAI, Hugging Face, Ollama, and Mistral.
- **Kotlin support**: Enhanced coroutine integration and virtual thread compatibility.
- **Post-GA**: Version 1.0.1 followed with bug fixes and minor enhancements.

### 🏗 Jakarta EE 11  
Jakarta EE 11 is in its final stages and is expected to be released in **June 2025**. This release focuses on modernization and alignment with Java 21:

- **Java 21 baseline**: Enables use of virtual threads, pattern matching, and other modern language features.
- **Updated specifications**: Including Jakarta Security 4.0, Jakarta RESTful Web Services, CDI, JSON Processing, Messaging, and more.
- **Platform simplification**: Continued removal of deprecated APIs and legacy constructs.
- **No new specifications introduced**: The focus is on stability and modernization, not expansion.

### ⚡ Quarkus 3.23.0  
Released May 28, 2025:
- **Named datasources for Hibernate Reactive**.
- **OIDC bearer step-up authentication**.
- **Camel Quarkus 3.23.0** integration.
- **GraalVM 25** early access support.

### 🌀 Micronaut 4.8.2  
While no new release dropped in May, version 4.8.2 (from April) remains current:
- **Performance and observability**: Updates to Micronaut Core, Micrometer, and JSON Schema modules.
- **Cloud-native focus**: Enhanced support for Azure Functions and AWS Lambda.

### 🌐 Helidon 4.2.0  
Helidon’s latest release includes:
- **Full virtual thread support**.
- **LangChain4j integration** for AI-native microservices.
- **Build-time DI**, GraalVM native image support, and MicroProfile APIs.

## ☕ Java Platform Update: JDK 25 in Focus

**JDK 25**, scheduled for General Availability in **September 2025**, is shaping up to be a substantial release. It builds on the innovations of JDK 21 through 24 and continues to refine Java’s performance, developer ergonomics, and platform capabilities.

### 📅 Key Milestones
- **Rampdown Phase One**: June 5, 2025  
- **Rampdown Phase Two**: July 17, 2025  
- **Initial Release Candidate**: August 21, 2025  
- **GA Release**: September 16, 2025

### ✅ JEPs Targeted for JDK 25

#### 🔄 Language & Developer Productivity
- JEP 507: Primitive Types in Patterns, `instanceof`, and `switch` (3rd Preview)  
- JEP 513: Flexible Constructor Bodies  
- JEP 512: Compact Source Files and Instance Main Methods  
- JEP 511: Module Import Declarations  

#### 🧵 Concurrency & Memory
- JEP 505: Structured Concurrency (5th Preview)  
- JEP 506: Scoped Values  
- JEP 519: Compact Object Headers  

#### 🧪 Performance & Profiling
- JEP 509: JFR CPU-Time Profiling (Experimental)  
- JEP 510: Key Derivation Function API  
- JEP 514: Ahead-of-Time Command-Line Ergonomics  
- JEP 515: Ahead-of-Time Method Profiling  
- JEP 518: JFR Cooperative Sampling  
- JEP 520: JFR Method Timing & Tracing  

#### 🧬 Platform & VM
- JEP 502: Stable Values (Preview)  
- JEP 508: Vector API (10th Incubator)  
- JEP 521: Generational Shenandoah GC  
- JEP 503: Remove the 32-bit x86 Port  
- JEP 470: PEM Encodings of Cryptographic Objects (Preview)  

## 🧠 Developer Trends

- **AI-first Java**: Spring AI and LangChain4j are leading the charge in LLM integration.
- **Virtual Threads**: Now supported across Spring, Helidon, and LangChain4j.
- **Cloud-native maturity**: Almost all major frameworks now offer native image support, OpenTelemetry integration, and Kubernetes-first tooling.

## 📅 What’s Next?

- **Jakarta EE 11** final release imminent.  
- **LangChain4j 1.1.0** roadmap includes agentic workflows and multi-modal support.  
- **Micronaut 4.9.x** and **Quarkus 3.24** expected in early summer.  
- **Spring Boot 3.5 GA** and **Spring Framework 7.0** are on the horizon.  
- **JDK 25** continues toward its September GA with early-access builds available.  
- **Jakarta EE 12** planning underway, with revived specs like Jakarta Portlet under discussion.
