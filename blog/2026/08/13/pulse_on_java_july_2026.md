# ☕ Pulse on Java – July 2026 Edition

July 2026 was a month of transition and momentum for the Java ecosystem. While JDK 27 entered its final stabilization phase ahead of its September release, the future of the platform became clearer as Project Valhalla's long-awaited Value Objects moved into JDK 28. Security also took center stage, with Oracle announcing a shift toward more frequent Java security updates. Meanwhile, AI continued its rapid integration into the Java world through Jakarta Agentic AI, GraalVM innovations, and growing ecosystem support across frameworks and libraries.

Here's a look at the most important developments from the Java community in July 2026.

| Category | Key Updates |
|-----------|------------|
| Java SE & OpenJDK | JDK 27 entered Rampdown Phase Two; JEP 401 (Value Objects) targeted to JDK 28; JEP 539 integrated; JEP 540 (Simple JSON API) proposed |
| Security | July Critical Patch Update; Oracle announces more frequent Java security updates; OpenJDK vulnerability advisory |
| Jakarta EE & Enterprise Java | Jakarta Agentic AI 1.0.0-M1; Open Liberty 26.0.0.7; Apache TomEE 10.2.0; Payara 7.2.0 |
| Frameworks & Libraries | Micronaut 5.1.0; Quarkus 3.38.0; JobRunr 8.8; LangChain4j 1.18; Java Operator SDK 5.5 |
| Tools & Infrastructure | GraalVM 25.2; Maven 4.0.0 RC6; TornadoVM releases; Native Image enhancements |
| Community & Events | Java conferences worldwide; Java Annotated Monthly; continued focus on AI, cloud-native development, and platform evolution |

---

## Java SE & OpenJDK

### JDK 27 enters Rampdown Phase Two

The most significant platform milestone in July was **JDK 27 entering Rampdown Phase Two on July 16**, effectively freezing the release's feature set and shifting the focus entirely to stabilization, testing, and bug fixing.

JDK 27 is scheduled for General Availability on September 15, 2026, and includes several notable features such as:

- Structured Concurrency
- Compact Object Headers enabled by default
- Lazy Constants
- Post-Quantum TLS Hybrid Key Exchange
- Java Flight Recorder Redaction
- Vector API improvements
- PEM Encodings support

With the feature window closed, the Java community can now focus on evaluating the release ahead of its September launch.

### Value Objects arrive in JDK 28

The biggest future-looking announcement came from Project Valhalla.

**JEP 401: Value Objects** was officially targeted to JDK 28 on July 30. Value Objects represent one of the largest changes to the Java object model since the language's inception.

Unlike traditional objects, Value Objects:

- Have no identity
- Are immutable by design
- Are compared by their values
- Enable more efficient memory layouts
- Improve locality and garbage collection performance

This work lays the foundation for a new generation of high-performance Java applications while preserving Java's familiar programming model.

### JEP 539: Strict Field Initialization

Closely related to Value Objects, **JEP 539** was integrated into JDK 28 during July.

The proposal strengthens JVM guarantees around field initialization, preventing code from observing uninitialized fields with default values such as `0` or `null`.

This work improves platform integrity and supports future Valhalla features.

### A standard JSON API on the horizon?

July also saw progress on **JEP 540: Simple JSON API**, which entered the Candidate stage.

If ultimately accepted, the proposal would provide a lightweight JSON API directly within the JDK, reducing the need for external libraries for common JSON processing tasks.

---

## Security & Operations

### Oracle announces more frequent Java security updates

One of the most important operational announcements of the month came from Oracle.

For decades, Java security fixes have largely followed the quarterly Critical Patch Update (CPU) cycle. Oracle announced plans to begin delivering Java security updates more frequently, with additional monthly releases planned beyond the traditional quarterly schedule.

For organizations running large Java estates, this change may require updates to:

- Patch management processes
- Vulnerability response procedures
- Deployment automation
- Testing pipelines

### July 2026 Critical Patch Update

Oracle's July Critical Patch Update was notable not only for Java but across Oracle's entire product portfolio.

The update included:

- 1,449 security patches
- 1,434 unique CVEs
- Coverage across 334 Oracle products

Java updates were released for:

- JDK 26.0.2
- JDK 25.0.4
- JDK 21.0.12
- JDK 17.0.20
- JDK 11.0.32
- JDK 8u501

### OpenJDK Vulnerability Advisory

The OpenJDK project also published its July security advisory, recommending upgrades across supported Java versions.

The advisory reinforced the importance of maintaining current JDK versions as organizations prepare for increasingly rapid security update cycles.

---

## Jakarta EE & Enterprise Java

### Jakarta Agentic AI takes its first major step

Perhaps the most exciting enterprise Java story of the month was the emergence of **Jakarta Agentic AI**.

The project released its first milestone, **Jakarta Agentic AI 1.0.0-M1**, with the goal of providing vendor-neutral APIs for building and running AI agents on Jakarta EE platforms.

The initiative aims to standardize:

- Agent programming models
- Agent lifecycle management
- LLM integrations
- Enterprise deployment patterns

As AI adoption accelerates, Jakarta Agentic AI may become one of the most important new specifications within the Jakarta ecosystem.

### Eclipse Starter adds Jakarta EE 11 support

The Eclipse Starter for Jakarta EE was updated to support Jakarta EE 11 runtimes, making it easier for developers to bootstrap modern enterprise applications targeting:

- Open Liberty
- GlassFish
- Payara

### Open Liberty 26.0.0.7

Open Liberty's July release focused on security and operational improvements, including:

- Logged-out SSO cookie tracking enabled by default
- Improved health endpoint configuration
- Platform maintenance updates

### Apache TomEE 10.2.0

Apache TomEE 10.2.0 was released with:

- Jakarta EE 10 support
- MicroProfile 6.1 support
- Java 17 baseline

### Payara 7.2.0

Payara delivered a significant platform update with:

- Jakarta EE 11 certification
- MicroProfile 6.1 support
- Security fixes
- Updated runtime components

---

## Frameworks & Libraries

### Micronaut 5.1.0

Micronaut released version 5.1.0, bringing updates across many of its ecosystem projects.

Areas receiving notable enhancements included:

- AI integrations
- Cloud support
- Data access
- Messaging
- Security
- Distributed tracing

Micronaut continues to position itself as a modern, cloud-native framework optimized for microservices and serverless deployments.

### Quarkus 3.38.0

Quarkus continued its rapid release cadence with version 3.38.

Highlights included:

- Weight-based Hibernate second-level cache eviction
- HTTP Problem Details support
- Dependency upgrades
- Performance and stability improvements

### JobRunr 8.8

JobRunr introduced several improvements, including:

- Better Quarkus integration
- Kotlin 2.4 support
- Performance optimizations
- Enhanced logging capabilities

### LangChain4j continues rapid evolution

LangChain4j maintained its fast pace with multiple July releases, further strengthening Java's growing AI ecosystem.

The project remains one of the leading choices for integrating LLMs and AI workflows into Java applications.

### Java Operator SDK 5.5

Cloud-native Java developers received Java Operator SDK 5.5, bringing additional capabilities for building Kubernetes Operators using Java.

---

## Tools & Infrastructure

### GraalVM 25.2

July's standout tooling release was **GraalVM 25.2**.

Key enhancements included:

- Graal Script Agent
- Native Image compressed references
- G1 GC support across all Native Image platforms
- Java Vector API enabled by default when available

The Native Image improvements were particularly noteworthy, with benchmarks showing substantial reductions in memory usage for certain workloads.

### Maven 4.0 RC6

The Maven team published **Maven 4.0.0 RC6**, moving the ecosystem another step closer to the long-awaited Maven 4 General Availability release.

The release focused on:

- Compatibility validation
- Plugin ecosystem readiness
- Stability improvements

### TornadoVM

TornadoVM continued pushing Java into high-performance and accelerated computing.

July releases added:

- Improved CUDA support
- Tensor Core optimizations
- FP8 and BF16 enhancements
- Updated JDK compatibility

### GPULlama3.java

An interesting development highlighted during July was the release of **GPULlama3.java**, demonstrating GPU-accelerated Llama 3 inference powered by TornadoVM and Java.

The project illustrates how Java is increasingly participating in AI and accelerated computing workloads traditionally dominated by Python.

---

## Community & Industry Highlights

### Java events around the world

July featured a busy conference calendar, including:

- WeAreDevelopers World Congress (Berlin)
- Java Forum Stuttgart
- ÜberConf
- JConf Dominicana
- JCRETE

These events reflected the continued strength and diversity of the global Java community.

### AI dominates community discussions

A recurring theme throughout July was the growing convergence of Java and AI.

Topics appearing across blogs, conference talks, framework releases, and community roundups included:

- Agentic AI architectures
- Enterprise AI governance
- LLM integrations
- AI-assisted development
- Security considerations for AI systems

Unlike previous years where AI experimentation often happened outside traditional enterprise Java stacks, July demonstrated increasing efforts to bring AI into established Java frameworks, runtimes, and standards.

---

## Final Thoughts

July 2026 was not defined by a single headline release. Instead, it was a month where the next chapter of Java became visible.

JDK 27 moved toward completion, while JDK 28 began taking shape around Value Objects and stronger platform integrity. Security update practices evolved toward a faster cadence, signaling new operational realities for Java teams. At the same time, Jakarta Agentic AI, GraalVM, Micronaut, Quarkus, and numerous ecosystem projects demonstrated that Java is embracing AI without abandoning its traditional strengths in reliability, portability, and enterprise development.

If June was about momentum, July was about direction. The platform's future is becoming clearer, and the Java ecosystem appears well-positioned for the challenges and opportunities ahead.