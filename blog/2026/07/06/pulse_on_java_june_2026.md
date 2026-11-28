# ☕ Pulse on Java – June 2026 Edition

*Your AI-generated monthly roundup of Java platform, framework, and community updates.*

**June 2026** was a month where several threads from May turned into concrete releases. The delayed **Spring** wave landed, **JDK 27** and **JDK 28** kept moving through early-access builds, **Jakarta EE 12** continued its steady specification work, and enterprise runtimes such as **GlassFish**, **TomEE**, **Open Liberty**, **Payara**, and **WildFly** all showed signs of active alignment around the latest Jakarta EE generation. Security also stayed unusually visible, with notable updates across **Spring Framework**, **Quarkus**, **Open Liberty**, and **Payara**. The signal across the month: Java is not just evolving — it is tightening its operational discipline around releases, CVEs, tooling, and governance.

Below is a summary of the key developments by category, followed by more detailed highlights.

| Category | Key Updates (June 2026) |
| --- | --- |
| **Java SE & JDK** | **JDK 27** EA builds progressed, **JDK 28** reached early builds, and **jtreg 8.3.0** added support for [JEP 512](https://openjdk.org/jeps/512). OpenJDK vs. GraalVM AI policy differences surfaced. |
| **Enterprise Java** | Jakarta EE 12 progressed; **[GlassFish 8.0.3](https://glassfish.org/download_gf8.html)**, **[TomEE 11.0.0-M1](https://tomee.apache.org/11.0.0-M1/release-notes.html)**, **[Open Liberty 26.0.0.6](https://openliberty.io/blog/2026/06/16/26.0.0.6.html)**, **[Payara June release](https://www.azul.com/blog/whats-new-in-azul-payara-june-2026/)**, and **[WildFly 41 Beta](https://www.wildfly.org/news/2026/06/26/New-WildFly-41-Beta-release/)** shipped updates. |
| **Frameworks & Libraries** | **[Spring Boot 4.1](https://spring.io/blog/2026/06/10/spring-boot-4)**, **[Spring AI 2.0.0](https://spring.io/blog/2026/06/12/spring-ai-2-0-0-GA-available-now/)**, **[Spring Framework 7.0.8 / 6.2.19](https://spring.io/blog/2026/06/08/spring-framework-7-0-8-and-6-2-19-available-now/)**, **[Spring Tools 5.2.0](https://spring.io/blog/2026/06/15/spring-tools-5-2-0-released/)**, **[Quarkus 3.37](https://quarkus.io/blog/quarkus-3-37-released/)**, **[Micronaut 5.0.3](https://micronaut.io/2026/06/30/micronaut-framework-5-0-3-released/)**, **[Hibernate ORM 8 Beta1](https://in.relation.to/2026/06/16/orm-80-beta1/)**. |
| **Tools & Infrastructure** | **[Gradle 9.6.0](https://docs.gradle.org/9.6.0/release-notes.html)**, **[Maven Surefire 3.6.0-M1](https://maven.apache.org/surefire/maven-surefire-plugin/whats-new-3-6-0.html)**, **[Eclipse IDE 2026-06](https://eclipseide.org/release/noteworthy/2026-06/)**, **[IntelliJ IDEA 2026.1.3](https://blog.jetbrains.com/idea/2026/06/intellij-idea-2026-1-3/)**. |
| **Community & Trends** | **[Commonhaus](https://www.commonhaus.org/activity/315.html)** growth, **[Open Source Sustainability Initiative](https://www.herodevs.com/blog-posts/herodevs-joins-commonhaus-foundation-open-source-sustainability-initiative-as-founding-member)**, and **[Devoxx Poland 2026](https://devoxx.pl/)**. |

## 🔬 Java SE & Platform Updates

### JDK 27 and JDK 28 Stayed Active

The Java SE platform story in June was steady. **JDK 27** continued through early-access builds, while **[JDK 28 EA](https://jdk.java.net/28/)** reached Build 4. This is classic OpenJDK cadence: visible progress, but mostly focused on stabilization.

A notable tooling update was **jtreg 8.3.0**, adding support for **[JEP 512](https://openjdk.org/jeps/512)** and improving preview/native test support. Not flashy, but critical for enabling newer language features to be tested reliably.

### AI Policy Became Part of the Platform Story

**OpenJDK’s contribution policy** now explicitly disallows AI-generated contributions, while still allowing private AI-assisted usage. In contrast, **GraalVM** allows AI-assisted contributions under human review. This difference highlights a growing ecosystem concern: provenance, IP, and review burden in an AI-assisted world.

## ☁️ Jakarta EE & Enterprise Java

### Jakarta EE 12 Kept Moving

**[Jakarta EE](https://jakarta.ee/)** 12 remained on track, with Core Profile specs advancing through milestones and broader platform specs progressing steadily. Configuration discussions (including potential Jakarta Config alignment) also resurfaced.

### Runtime and Implementation Updates

- **[GlassFish 8.0.3](https://glassfish.org/download_gf8.html)** improved security and performance, including faster startup and better Faces rendering.
- **[TomEE 11.0.0-M1](https://tomee.apache.org/11.0.0-M1/release-notes.html)** targeted Jakarta EE 11 and MicroProfile 7.1, clearly marked as a milestone.
- **[Open Liberty 26.0.0.6](https://openliberty.io/blog/2026/06/16/26.0.0.6.html)** fixed CVEs and updated documentation.
- **[Payara June 2026 release](https://www.azul.com/blog/whats-new-in-azul-payara-june-2026/)** patched CSRF/SSRF issues and continued Jakarta EE 11 alignment.
- **[WildFly 41 Beta](https://www.wildfly.org/news/2026/06/26/New-WildFly-41-Beta-release/)** introduced improvements in shutdown handling, OIDC, TLS, and bootable JARs.

## 🚀 Major Framework Releases & Ecosystem News

### Spring’s Delayed Wave Landed

June was a big month for Spring:

- **[Spring Boot 4.1](https://spring.io/blog/2026/06/10/spring-boot-4)** added gRPC auto-config, SSRF mitigations, Kotlin 2.3 support, and improved observability.
- **[Spring AI 2.0.0](https://spring.io/blog/2026/06/12/spring-ai-2-0-0-GA-available-now/)** aligned with Boot 4 / Framework 7 and expanded MCP and tool-calling support.
- **[Spring Framework 7.0.8 / 6.2.19](https://spring.io/blog/2026/06/08/spring-framework-7-0-8-and-6-2-19-available-now/)** addressed a large set of CVEs and effectively closed the 6.2.x OSS line.
- **[Spring Tools 5.2.0](https://spring.io/blog/2026/06/15/spring-tools-5-2-0-released/)** added AI-centric tooling (Claude plugin, MCP support, Spring AI indexing).

### Quarkus Balanced Features and Security

- **[Quarkus 3.37](https://quarkus.io/blog/quarkus-3-37-released/)** introduced experimental `jlink` packaging and upgraded Hibernate components.
- **[CVE-2026-50559 advisory](https://quarkus.io/blog/CVE-2026-50559/)** triggered emergency releases across supported versions.

### Other Libraries and Frameworks

- **[Micronaut 5.0.3](https://micronaut.io/2026/06/30/micronaut-framework-5-0-3-released/)** continued stabilization across modules.
- **[Hibernate ORM 8.0.0.Beta1](https://in.relation.to/2026/06/16/orm-80-beta1/)** added Jakarta Persistence 4.0 support and new data features.
- **[LangChain4j 1.17.0](https://github.com/langchain4j/langchain4j/releases/tag/1.17.0)** expanded agent patterns and integrations.
- **[Hardwood 1.0](https://www.morling.dev/blog/hardwood-1-0-fast-lightweight-apache-parquet-reader-for-the-jvm/)** delivered a lightweight Parquet reader.
- **[Endive 1.0](https://endive.run/blog/endive-1.0/)** introduced a JVM-native WebAssembly runtime.

## 🛠️ Developer Tools & Infrastructure

- **[Gradle 9.6.0](https://docs.gradle.org/9.6.0/release-notes.html)** improved Configuration Cache and CI ergonomics.
- **[Maven Surefire 3.6.0-M1](https://maven.apache.org/surefire/maven-surefire-plugin/whats-new-3-6-0.html)** unified test execution on JUnit Platform.
- **[Eclipse IDE 2026-06](https://eclipseide.org/release/noteworthy/2026-06/)** shipped with 60+ coordinated projects.
- **[IntelliJ IDEA 2026.1.3](https://blog.jetbrains.com/idea/2026/06/intellij-idea-2026-1-3/)** focused on stability and developer experience fixes.

## 🌐 Community & Industry Highlights

- **[OkHttp, Okio, Retrofit, SQLDelight joined Commonhaus](https://www.commonhaus.org/activity/315.html)**, strengthening neutral governance for key JVM libraries.
- **[HeroDevs joined the Open Source Sustainability Initiative](https://www.herodevs.com/blog-posts/herodevs-joins-commonhaus-foundation-open-source-sustainability-initiative-as-founding-member)** to support long-term maintenance and CVE patching.
- **[Devoxx Poland 2026](https://devoxx.pl/)** gathered ~2,700 attendees and highlighted Java, AI, and cloud-native trends.

## 🔒 Security Updates

Security remained a defining theme:

- **[Spring Framework 7.0.8 / 6.2.19](https://spring.io/blog/2026/06/08/spring-framework-7-0-8-and-6-2-19-available-now/)** fixed multiple CVEs.
- **[Quarkus CVE-2026-50559](https://quarkus.io/blog/CVE-2026-50559/)** required urgent upgrades.
- **[Open Liberty 26.0.0.6](https://openliberty.io/blog/2026/06/16/26.0.0.6.html)** addressed CVEs.
- **[Payara June release](https://www.azul.com/blog/whats-new-in-azul-payara-june-2026/)** patched Admin Console vulnerabilities.

## Bottom Line & Outlook

**June 2026 was a delivery month.** The Spring stack landed, Jakarta EE kept progressing, runtimes aligned, and tooling improved in meaningful ways. At the same time, security and governance became more visible — from CVE response speed to AI contribution policies.

If May showed direction, **June showed execution discipline** — and that is exactly what keeps the Java ecosystem strong at scale.

## Sources

This edition is based on authoritative material published during June 2026, including OpenJDK and Inside Java updates, Eclipse Foundation Jakarta EE tracking pages, GlassFish, TomEE, Open Liberty, Payara, and WildFly release announcements, Spring, Quarkus, Micronaut, and Hibernate blog posts, Gradle, Maven, Eclipse IDE, and IntelliJ IDEA release notes, Commonhaus and HeroDevs community updates, and InfoQ’s weekly Java news roundups.