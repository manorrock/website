# ☕ Pulse on Java – October 2025 Edition

*Your AI-generated monthly roundup of Java framework and platform updates. Please direct your comments directly to us at blog@manorrock.com*

Welcome to the October 2025 edition of **Pulse on Java**, where we recap the key developments, announcements, and trends across the Java ecosystem during the month. **October was a significant month** with Oracle's quarterly security CPU update delivering critical patches across multiple JDK versions, important framework milestones including Spring Boot 4 release candidates and Micronaut 4.10, and important tooling updates from Gradle and IntelliJ IDEA. Let's dive into the details.

## 🚀 Java Platform Updates: October CPU & JDK 26 Preview

**Oracle's October 2025 Critical Patch Update (CPU) was released**, delivering important security fixes and updates across all supported JDK versions including **JDK 25.0.1, 21.0.9, 17.0.17, 11.0.29, and 8u471**[1](https://github.com/adoptium/temurin25-binaries/releases). This quarterly update addresses multiple security vulnerabilities and includes several notable changes that Java developers should be aware of.

Key highlights from the October CPU include:

*   *Security Patches:* The update addresses numerous security vulnerabilities across the Java platform. Independent security analysis emphasized the CPU's breadth, covering hundreds of CVEs across Oracle products[2](https://maven.apache.org/docs/4.0.0-rc-4/release-notes.html). For Java teams, the priority is to **roll out the JDK updates** promptly and **audit XPath usage** to avoid potential outages from new security hardening measures[3](https://newreleases.io/project/github/micronaut-projects/micronaut-core/release/v4.10.0).

*   *Platform-Specific Issues:* Developers should note an **IPv6 datagram packet loss issue on macOS 15.6+**; a temporary workaround is using the flag `-Djava.net.preferIPv4Stack=true`[1](https://github.com/adoptium/temurin25-binaries/releases). Additionally, the **cacerts certificate store removed four AffirmTrust roots**. If your application processes XML with XPath and **secure processing enabled**, you may encounter failures on external DTDs—it's recommended to retest downstream pipelines[1](https://github.com/adoptium/temurin25-binaries/releases).

*   *Vendor Distributions:* Multiple OpenJDK distributions released aligned builds for the October CPU. **Eclipse Temurin (Adoptium)** published **25.0.1+8** and aligned updates across LTS lines, though **Windows AArch64 for JDK 25 remains unavailable** this cycle due to test failures[4](https://adoptium.net/news/2025/11/eclipse-temurin-8u472-11029-17017-2109-2501-available). **Microsoft Build of OpenJDK** shipped versions **25.0.1 / 21.0.9 / 17.0.17 / 11.0.29** with full platform support[5](https://devblogs.microsoft.com/java/java-openjdk-oct-2025-patch-security-update/). **Azul Zulu** delivered their October quarterly update across all major versions[6](https://docs.azul.com/core/release/october-2025/release-notes).

*   *GraalVM Update:* **GraalVM 25.0.1** was released aligned with the CPU, with an important deprecation notice: **macOS x64 is deprecated** and 25.0.1 is the last version supporting it[7](https://docs.oracle.com/en/graalvm/jdk/25/docs/release-notes/). The release includes compiler and native-image fixes along with diagnostic improvements[7](https://docs.oracle.com/en/graalvm/jdk/25/docs/release-notes/).

### Looking Ahead: JDK 26

**JDK 26 development continued with several JEPs proposed or targeted** for the March 17, 2026 GA release[8](https://micronaut.io/):

*   **JEP 525: Structured Concurrency (6th preview)** continues to refine the API for treating groups of related threads as a unit[8](https://micronaut.io/).
*   **JEP 529: Vector API (11th incubator)** advances SIMD capabilities for math-intensive operations[8](https://micronaut.io/).
*   **JEP 516: AOT Object Caching with Any GC** aims to reduce startup and warmup times across different garbage collectors[8](https://micronaut.io/).
*   **JEP 500: Prepare to Make Final Mean Final** will end deep-reflection writes to `final` fields[8](https://micronaut.io/).
*   **JEP 517: HTTP/3 in the HTTP Client** brings modern HTTP/3 protocol support to the standard library[8](https://micronaut.io/).

For ongoing updates and community discussions around JDK 26 features like **HTTP/3** and other enhancements, the **Dev.java** feed remains a valuable resource[9](https://gradle.org/releases/).

<!-- Copilot-Researcher-Visualization -->

## 🔧 Framework & Library Updates

October saw major activity across popular Java frameworks, with several reaching important milestones and release candidates as they prepare for production releases aligned with JDK 25 and Jakarta EE 11.

*   *Spring Ecosystem:* The **Spring Boot 4.0.0 release candidates** (RC1 and RC2) shipped in October, representing the culmination of the Spring team's work on the next major version[10](https://docs.gradle.org/9.2.0/release-notes.html). RC1/RC2 complete the framework's **modularization**, add a new **`RestTestClient`** for testing REST clients, deliver **Redis observability improvements**, refactor tracing modules, and (in RC2) restore **Jersey support**[10](https://docs.gradle.org/9.2.0/release-notes.html). Alongside this, **Spring Framework 7.0.0-RC1** established its baseline as **JDK 17+** (with JDK 25 recommended), **Jakarta EE 11**, **Kotlin 2.2**, and **GraalVM 25**, while adding resiliency annotations and context propagation refinements[11](https://gradle.org/releases/). Developers planning to migrate should review the RC release notes for API removals and deprecations.

*   *Micronaut 4.10.0:* The Micronaut team released **version 4.10** with exciting new capabilities[12](https://www.jetbrains.com/idea/nextversion/). This release introduces the **Micronaut MCP (Model Context Protocol)** module for AI integrations, delivers updates across Netty and servlet stacks, adds **Loom carrier support** for virtual-thread workloads, and includes security and validation enhancements[12](https://www.jetbrains.com/idea/nextversion/). For teams already using virtual threads, evaluating the MCP integrations and Loom carrier paths can unlock performance improvements in concurrent scenarios.

*   *Quarkus 3.29:* Red Hat's Quarkus reached **version 3.29** with two notable additions[13](https://blog.jetbrains.com/en/idea/2025/09/intellij-idea-2025-3-eap). First, **support for multiple cache backends** gives developers flexibility in choosing caching strategies for their applications. Second, **Qute DAP (Debug Adapter Protocol) debugger support** enables IDE-based debugging of Qute templates, making template development significantly more productive[13](https://blog.jetbrains.com/en/idea/2025/09/intellij-idea-2025-3-eap). The release followed its regular schedule with CR1 mid-month and the final release on October 29th.

*   *Grails 7.0 GA:* A major milestone for the Groovy web framework community: **Grails 7.0 reached General Availability** and simultaneously became a **Top-Level Apache Project**[14](https://eclipseide.org/release/noteworthy/). Grails 7.0 upgrades to **Java 17+ baseline**, **Spring Boot 3.5**, and **Jakarta EE 10**, while adding Software Bill of Materials (SBOM) support, reproducible builds, and testing improvements[14](https://eclipseide.org/release/noteworthy/). For Grails users, this represents a significant modernization of the platform and stronger community governance under the Apache umbrella.

*   *Enterprise Server Updates:* On the Jakarta EE server front, **Open Liberty 25.0.0.10** (released October 7) added **override library support** in the application classloader for surgical hotfixing without full rebuilds, plus **Java 25 compatibility** and several CVE fixes[15](https://openliberty.io/blog/2025/10/07/25.0.0.10.html). **WildFly 38** progressed from **Beta 1** (October 2) to **Final** (October 16), delivering **MicroProfile 7.1** and **Jakarta EE 11 integration in WildFly Preview**, along with a migration tool for upgrading from WildFly 28[16](https://www.wildfly.org/news/2025/10/02/New-WildFly-38-Beta-release/). **Apache Tomcat 11.0.x** advanced through several releases including **11.0.13** mid-month, adding CIDR support, proxy updates, and AJP fixes, with security notes highlighting multipart upload temp-file cleanup hardening[17](https://tomcat.apache.org/tomcat-11.0-doc/changelog.html).

*   *Build Tool Evolution:* **Gradle 9.2** launched on October 29 as a significant update[18](https://docs.gradle.org/9.2.0/release-notes.html). The release delivers first-class **Windows on ARM (ARM64) support**, faster time-to-first-task when hitting the configuration cache, improved publishing APIs, and promotes the **Daemon toolchain to stable**[18](https://docs.gradle.org/9.2.0/release-notes.html). Teams running Windows on ARM can now expect native performance, while all Gradle users will see build start latency improvements with configuration cache hits. **Maven 4** remains in RC stage (RC-4 shipped in June); developers can use the new **Maven Upgrade Tool (`mvnup`)** to prepare POMs for Maven 4's consumer/build POM split and Java 17 runtime requirement[19](https://maven.apache.org/docs/4.0.0-rc-4/release-notes.html).

*   *IDE Updates:* **IntelliJ IDEA 2025.3 EAP** opened in mid-September and continued with multiple October builds[20](https://blog.jetbrains.com/en/idea/2025/09/intellij-idea-2025-3-eap). This cycle introduces a **unified distribution** (more features available for free), and delivers **full support for Java 25, Spring Boot 4, and JUnit 6**[20](https://blog.jetbrains.com/en/idea/2025/09/intellij-idea-2025-3-eap). For developers eager to pilot Java 25 and Spring Boot 4, the EAP provides early access, though with the usual stability caveats of pre-release software. **Eclipse IDE 2025-09** (released September 9) brought Java tooling and platform updates detailed in the release train's "New & Noteworthy" documentation[14](https://eclipseide.org/release/noteworthy/).

## 🏃‍♂️ Community & Events Highlights

October's Java community buzz centered on security updates, framework migrations, and knowledge sharing:

*   *Security Focus:* With the October CPU delivering critical patches, the community rallied around rapid adoption. Security researchers published analyses of the CPU's breadth, and practitioners shared XPath hardening strategies and workarounds for the macOS IPv6 datagram issue. The emphasis on security underscores the Java community's commitment to maintaining a secure platform.

*   *Learning Resources:* **Java Annotated Monthly (October 2025)** compiled a compact digest featuring Java 25 highlights, tutorials, and conference talks—an excellent resource for teams looking to upskill on the latest platform features[21](https://gradle.org/whats-new/gradle-9/). Community blogs and video content focused heavily on the October CPU migration paths, Spring Boot 4 RC features, and Gradle 9.2's Windows ARM support.

*   *JavaFX Discussions:* A long-simmering topic resurfaced this month: **JavaFX reintegration** into the OpenJDK discussions[22](https://github.com/micronaut-projects/micronaut-aws/releases/tag/v4.10.0). Oracle indicated it's exploring ways to make JavaFX more approachable alongside the JDK, potentially lowering barriers for desktop Java development. While no concrete plans were announced, the renewed attention signals community interest in simplified JavaFX distribution.

*   *Dev.java Posts:* The official **Dev.java** site published heads-ups for **JDK 26 HTTP/3 support** and progress on **Project Valhalla's Value Classes**[9](https://gradle.org/releases/). For teams piloting JDK 26 early-access builds, these posts provide valuable context on upcoming features and migration considerations.

## 📅 Looking Ahead

As we close out October and head into the final months of 2025, several trends and upcoming milestones deserve attention:

*   *Spring Boot 4 GA:* With RC2 shipped, the Spring team is on track for a **Spring Boot 4.0 General Availability** release in the coming weeks. Teams should begin planning migration timelines, reviewing breaking changes, and testing applications against the RCs.

*   *JDK 26 Early Access:* The **JDK 26 project page** shows a GA target of **March 17, 2026**[8](https://micronaut.io/). With JEPs like HTTP/3 support, AOT object caching, and "make final mean final" on the horizon, early adopters should start testing EA builds to provide feedback and prepare for the upcoming release.

*   *Jakarta EE 11 Adoption:* With WildFly 38, Open Liberty 25.0.0.10, and ongoing GlassFish 8 milestones all targeting Jakarta EE 11, we're seeing the enterprise Java ecosystem converge on the new specification. Expect more application servers and frameworks to announce Jakarta EE 11 compatibility through year-end.

*   *Security Patch Cycle:* The next Oracle CPU is scheduled for January 2026. Teams should plan to allocate time for testing and deployment, especially given the October CPU's XPath hardening changes that required careful regression testing.

*   *Conference Season Wrap-Up:* Fall 2025 conferences are wrapping up, with recorded sessions and highlights beginning to circulate. These materials—covering Java 25, virtual threads, security best practices, and framework migrations—will provide valuable learning opportunities for teams unable to attend in person.

**In summary**, October 2025 was a month of consolidation and preparation: critical security updates were deployed, major frameworks reached important release milestones, and the community began looking ahead to JDK 26 and Jakarta EE 11 adoption. The emphasis on security, coupled with exciting framework innovations and tooling improvements, positions the Java ecosystem well as we head into the final quarter of the year.
