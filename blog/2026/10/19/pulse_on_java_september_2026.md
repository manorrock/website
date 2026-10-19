# ☕ Pulse on Java – September 2026 Edition

*A monthly roundup of the Java platform, enterprise stack, frameworks, tooling, and community.*

**September 2026 was a release month in every sense of the word.** [JDK 27 reached general availability](https://openjdk.org/projects/jdk/27/), JavaFX shipped its matching release, Jakarta CDI 5.0 crossed the finish line, Quarkus published a new LTS, and major framework and build-tool projects moved quickly to support the new Java baseline.

There was also plenty happening just beyond the headline releases. OpenJDK began revealing the direction of JDK 28, Eclipse Temurin strengthened its reproducible-build story, Micronaut completed its move to the Commonhaus Foundation, and new work around agent-to-agent communication and checkpoint-and-restore started taking shape in the Jakarta ecosystem.

Below is a summary of the key developments by category, followed by more detailed highlights.

| Category | Key Updates (September 2026) |
| --- | --- |
| **Java SE & JDK** | **JDK 27** reached GA with nine JEPs, including compact object headers by default, G1 as the default collector everywhere, post-quantum hybrid TLS key exchange, and continued previews of structured concurrency, primitive patterns, lazy constants, and PEM encoding. Work on **JDK 28** also moved forward around ahead-of-time compilation and ZGC. |
| **Enterprise Java** | **Jakarta CDI 5.0** completed its release review, while **Jakarta CRaC** entered community review. **A2A Jakarta 1.0.0.Final** brought the A2A protocol to Jakarta-compatible runtimes, and Payara delivered MicroProfile 7.1, security fixes, and broader access to its Jakarta Agentic AI preview. |
| **Frameworks & Libraries** | Spring delivered another broad milestone wave, **Quarkus 3.40** became the project's latest LTS release, and **Micronaut 5.2** arrived shortly after Micronaut confirmed its new home at the Commonhaus Foundation. **Apache Groovy 6.0** also reached GA. |
| **Tools & Infrastructure** | **JavaFX 27**, **Gradle 9.8**, and **Maven 4.0.0-rc-7** kept the development stack moving with Java 27. **TornadoVM 7.0** added a tile-oriented GPU programming model, while Eclipse Temurin expanded independently verifiable reproducible builds. |
| **Community & Trends** | IntelliJ IDEA Conf, Java Forum Nord, and JakartaOne Livestream Chinese all took place during September. Across releases and events, the recurring themes were startup performance, AI integration, supply-chain trust, and Java's continuing expansion into heterogeneous computing. |

## 🔬 Java SE & Platform Updates

### JDK 27 Reached General Availability

The biggest Java story of the month was an easy one to identify: [JDK 27 became generally available on September 15](https://mail.openjdk.org/archives/list/jdk-dev@openjdk.org/thread/ORGGLMN75HFEWP7YL3ZLGHLYHVIBJDYT/).

This is not an LTS release, but it is a substantial one. JDK 27 contains nine JEPs, and several of them change the platform's default behavior rather than simply adding APIs for adventurous developers to try.

Two runtime changes deserve particular attention. [JEP 523 makes G1 the default garbage collector in all environments](https://openjdk.org/jeps/523), removing the situations where HotSpot previously selected Serial GC automatically. [JEP 534 enables compact object headers by default](https://openjdk.org/jeps/534), reducing object-header size on supported 64-bit configurations. Together, these changes make JDK 27 interesting even for applications that do not adopt a single new language feature.

Security also received a major platform-level addition through [JEP 527, Post-Quantum Hybrid Key Exchange for TLS 1.3](https://openjdk.org/jeps/527). It combines established key-exchange algorithms with quantum-resistant ones, allowing applications using the standard JSSE APIs to benefit without rewriting their networking code. [JEP 536](https://openjdk.org/jeps/536) complements that work from an operational angle by adding in-process data redaction to Java Flight Recorder.

The language and API story remains more evolutionary:

* [Lazy Constants returned for a third preview](https://openjdk.org/jeps/531).
* [Primitive types in patterns, `instanceof`, and `switch` entered a fifth preview](https://openjdk.org/jeps/532).
* [Structured Concurrency reached a seventh preview](https://openjdk.org/jeps/533).
* [The Vector API continued in a twelfth incubator round](https://openjdk.org/jeps/537).
* [PEM encodings of cryptographic objects received a third preview](https://openjdk.org/jeps/538).

That may sound like a long time in preview, especially for structured concurrency, but it also shows the value of Java's deliberate release process. These APIs affect fundamental programming models and deserve careful iteration before becoming permanent.

For anyone planning an upgrade, the [JDK 27 release notes](https://www.oracle.com/java/technologies/javase/27-relnote-issues.html) are worth reading rather than treating this as a routine version bump. New garbage-collector defaults, compact headers, security changes, removals, and compatibility notes can matter even when application source remains unchanged.

### JDK 28 Work Started Moving Into View

JDK 27 shipped in the middle of the month, but OpenJDK work immediately continued toward JDK 28.

The most notable September movement came from [JEP 544, Ahead-of-Time Code Compilation](https://openjdk.org/jeps/544). It progressed from candidate status to proposed-to-target during the month and was targeted to JDK 28 at the end of the review period.

Rather than creating a separate static Java runtime, the proposal extends HotSpot's existing ahead-of-time cache. A training run can place optimized native code in that cache, allowing later production runs to start with useful compiled code already available. If the workload changes, the JVM can still deoptimize and generate new JIT-compiled code. That combination of faster startup and HotSpot's normal adaptability is what makes this work especially interesting.

Two ZGC proposals also became candidates on September 21. [JEP 545](https://openjdk.org/jeps/545) focuses on faster startup and warmup by improving how ZGC acquires and prepares memory. [JEP 546](https://openjdk.org/jeps/546) proposes adaptive heap sizing based on application demand, available resources, and neighboring workloads.

These were candidate JEPs during September, not delivered JDK 28 features. Still, the direction is clear: startup, warmup, and more adaptive resource use remain central platform priorities.

### Eclipse Temurin Strengthened Reproducible Builds

On September 2, Eclipse Adoptium published a substantial update to its [Temurin reproducible verification build program](https://adoptium.net/news/2026/09/temurin-reproducible-verification-builds-2026).

Eclipse Temurin JDK 21 and later builds are now described as fully reproducible across the project's major Linux, Windows, and macOS targets. Third parties can use the published scripts and build metadata to rebuild a Temurin binary from upstream sources, compare it byte for byte, and publish a signed CycloneDX attestation.

Successfully verified binaries can then receive a **"Reproduced by a third party"** badge on the Temurin download site. IBM and Red Hat supplied the first third-party attestations.

This is not the flashiest Java news of the month, but it is important work. Reproducible builds turn trust in a binary from "the vendor says this is what it built" into something an independent organization can verify and document.

## ☁️ Jakarta EE & Enterprise Java

### Jakarta CDI 5.0 Crossed the Finish Line

[Jakarta CDI 5.0](https://jakarta.ee/specifications/cdi/5.0/) completed its release review on September 11, making it one of the clearest signs of progress toward Jakarta EE 12.

The release raises the minimum Java version to 17 and adds several useful capabilities. `@Eager` supports eager initialization of application-scoped beans, `@AutoClose` can invoke `close()` automatically when an `AutoCloseable` bean is destroyed, and the new asynchronous invoker API expands what can be expressed through CDI. The specification also adds default beans through `@Reserve`, synthetic injection points, global scopes, and additional build-compatible extension support.

There is migration work to note. The CDI API and TCK Maven coordinates have changed, deprecated `BeanManager` methods were removed, and old Security Manager references are gone. Relocation artifacts should ease the API transition, but build files and extension integrations still deserve a careful review.

### Jakarta CRaC Entered Community Review

A new [Jakarta CRaC proposal](https://projects.eclipse.org/proposals/jakarta-crac) appeared in September, aiming to define standard behavior for checkpoint-and-restore events in Jakarta EE applications and runtimes.

The proposal is intentionally complementary to the existing CRaC API rather than a replacement for it. Its focus is on giving Jakarta EE implementations a common notification mechanism and allowing applications to identify resources that require attention around checkpoint and restore.

This could eventually make checkpoint-and-restore behavior more portable across enterprise runtimes, including environments using CRaC or IBM InstantOn. For now, though, it is a project proposal in community review—not a finished specification.

### Runtime and Implementation Updates

September also produced several noteworthy runtime developments:

* [A2A Jakarta 1.0.0.Final](https://www.wildfly.org/news/2026/09/10/A2A-Jakarta-1-0-0-Final-is-released/) delivered Jakarta integrations for the A2A Java SDK. It supports version 1.0 of the Agent2Agent protocol across JSON-RPC, gRPC, and HTTP+JSON/REST transports, while retaining a compatibility layer for 0.3 clients. The release also includes multi-tenancy support and an enterprise example that runs an agent across two WildFly nodes.
* The [September Azul Payara release](https://www.azul.com/blog/whats-new-in-the-september-2026-azul-payara-release/) brought MicroProfile 7.1 to the supported Payara 7 and 6 lines. Applications using MicroProfile OpenTracing need attention because that API has been removed in favor of the OpenTelemetry-based MicroProfile Telemetry model.
* Payara Community 7.2026.9 extended its Jakarta Agentic AI preview to Payara Micro and Payara Embedded. The specification remained at Milestone 1, so this is still an experimentation opportunity rather than a stable application contract.
* [Open Liberty 26.0.0.10 beta](https://openliberty.io/blog/2026/09/22/26.0.0.10-beta.html) added beta support for JDK 27, including guidance about the platform's new G1 default in constrained environments.

## 🚀 Major Framework Releases & Ecosystem News

### Spring

September was another busy milestone month across the Spring portfolio.

The clearest example was [Spring Boot 4.2.0-M2](https://spring.io/blog/2026/09/25/spring-boot-4-2-0-M2-available-now/). It added SSL bundle support for LDAP and embedded LDAPS, along with support for OpenTelemetry semantic conventions and common OTLP endpoint, header, and compression configuration.

Second milestones also moved through Spring Framework, Spring Security, Spring Data, Spring Batch, Spring Integration, Spring AMQP, Spring for Apache Kafka, and Spring for GraphQL. First milestones appeared for the next Spring AI, Spring Cloud, Spring Web Services, and Spring LDAP lines.

The individual changes vary, but the overall signal is straightforward: the Spring ecosystem is already well into the next development cycle, with observability and consistent platform configuration continuing to receive attention.

### Quarkus

Quarkus closed the month with [Quarkus 3.40 LTS](https://quarkus.io/blog/quarkus-3-40-released/) on September 30.

The new LTS is based on Quarkus 3.39 and supersedes Quarkus 3.33 as the project's current long-term support line. Quarkus supports its LTS releases for 12 months, making 3.40 the natural upgrade target for teams that prefer a slower-moving production baseline.

Most of the feature story comes from the releases between 3.33 and 3.39: JAR tree shaking, profile-guided optimization for native builds, embedded SBOMs, extension-based modularity, reflection-free Jackson serializers, Hibernate ORM 7.4, and post-quantum cryptography support in the TLS registry. Quarkus 3.40 packages that work into a stable support point and updates platform components including Camel Quarkus, Quarkus CXF, LangChain4j, the Quarkus MCP Server, Debezium, and the Operator SDK.

### Micronaut

Micronaut began September with an organizational milestone and ended it with a technical one.

On September 1, the project confirmed that [Micronaut is now a project of the Commonhaus Foundation](https://micronaut.io/2026/09/01/micronaut-new-home-at-commonhaus/). The move gives Micronaut a long-term, vendor-neutral home while leaving its repositories, Apache 2.0 license, contribution process, and technical direction with the project community.

Then, on September 27, [Micronaut Framework 5.2.0](https://micronaut.io/2026/09/27/micronaut-framework-5-2-0/) arrived with updates across a large part of the ecosystem. Highlights include experimental Micronaut for Python support through GraalPy, a new build-time Jakarta Expression Language implementation, upsert support in Micronaut Data, WebSocket support across the servlet runtimes, and broader serialization format support.

The Commonhaus move may be the longer-lasting development. Healthy governance and a durable project home are easy to overlook when release notes are arriving every week, but they matter greatly to organizations building long-lived applications on an open-source framework.

### Other Libraries and Frameworks

Two additional releases stood out:

* [Apache Groovy 6.0 reached GA](https://www.mail-archive.com/announce@apache.org/msg12006.html) after three release candidates. Along with the usual fixes and dependency updates, the release improves lexer and parser performance by simplifying several hot paths in Groovy's grammar processing.
* [TornadoVM 7.0](https://tornadovm.readthedocs.io/en/latest/CHANGELOG.html) arrived on September 22. Its headline addition is `TileContext`, a higher-level model for describing tiled GPU work while allowing NVIDIA's CUDA Tile compiler to select thread organization, tensor-core instructions, and shared-memory staging. The release also added a method-based task API and continued work across CUDA, OpenCL, and Metal.

## 🛠️ Developer Tools & Infrastructure

### JavaFX 27 Shipped Alongside the JDK

[JavaFX 27 became available on September 15](https://gluonhq.com/news/2026-09-15-javafx-27-is-now-available/), matching the JDK 27 release date.

Metal is now the default rendering pipeline on macOS, replacing the older OpenGL-based ES2 pipeline. CSS gained conditional stylesheet imports and media queries, controls can be placed in a window title bar, and `RichTextArea` added tab stops, embedded images, text backgrounds, and wavy underline attributes.

JavaFX 27 now requires JDK 25 or later. Teams with macOS applications should also give the new Metal default some deliberate testing, although an ES2 fallback remains available during migration.

### Gradle and Maven Kept Moving

[Gradle 9.8](https://docs.gradle.org/9.8.0/release-notes.html) added support for running Gradle itself on Java 27 and for using Java 27 through toolchains. It also introduced an option to reuse Maven mirror settings, improved build-authoring APIs, and included a substantial Windows performance improvement.

[Maven 4.0.0-rc-7](https://maven.apache.org/docs/4.0.0-rc-7/release-notes.html) delivered another round of fixes, documentation improvements, dependency updates, and performance work. The Maven team still labels the Maven 4 line as preview software rather than production-ready, but RC7 was presented as the expected final candidate before GA, subject to community feedback.

A smaller project worth watching is [Lathe](https://ag-libs.github.io/2026/09/27/lathe-a-java-language-server-from-your-maven-build/), an open-source Java language server introduced on September 27. Instead of independently reconstructing a Maven project model, Lathe captures class paths, module paths, annotation processing, testing, and launch information directly from a working Maven build. It is early, but the idea—make the editor follow the build rather than guess what the build means—is a compelling one.

## 🌐 Community & Industry Highlights

September offered a good mix of online and in-person community activity.

[IntelliJ IDEA Conf 2026](https://lp.jetbrains.com/intellij-idea-conf-2026/) ran online on September 8–9, with session recordings made available afterward. The event continued JetBrains' practical focus on IDE workflows and modern Java development.

[Java Forum Nord](https://javaforumnord.de/2026/) took place in Hannover on September 22, bringing together Java user groups from northern Germany for a community-led conference program.

On September 24, [JakartaOne Livestream Chinese](https://jakartaone.jakarta.ee/jakartaone-livestream-2026-chinese/) focused on Jakarta EE, cloud-native Java, data specifications, enterprise AI, and spec-driven development for AI-assisted engineering.

The common thread across these events matched the release activity: Java's community is increasingly discussing AI agents, specifications, observability, startup behavior, and developer tooling as connected concerns rather than separate trends.

## 🔒 Security Updates

September's security cadence needs a little clarification.

OpenJDK's normal vulnerability-fix schedule remains the third Tuesday of January, April, July, and October. There was no new regular OpenJDK advisory during September; the most recent OpenJDK advisory remained the out-of-cycle August 18 update.

Oracle did, however, publish a [September Critical Security Patch Update](https://www.oracle.com/security-alerts/cspusep2026.html) on September 15. Its Java SE risk matrix contained three remotely exploitable compiler vulnerabilities affecting supported Oracle GraalVM for JDK 17 and 21 releases, plus the older GraalVM Enterprise Edition line. This was a targeted CSPU, distinct from the usual cumulative OpenJDK quarterly update.

At the framework level, Payara patched a `jackson-databind` resource-exhaustion issue across every supported Payara branch. JavaFX 27 also included several security fixes in graphics, media, playlist handling, editor selection, and WebView resource loading.

The practical lesson is familiar: track the cadence of the JDK distribution, runtime, framework, and major dependencies separately. September showed that security updates do not always arrive through a single Java-wide channel.

## Bottom Line & Outlook

**September 2026 was one of the most consequential Java months of the year.** JDK 27 changed important runtime defaults, strengthened Java's post-quantum security story, and moved several long-running previews forward. Jakarta CDI 5.0 shipped, Quarkus established a new LTS baseline, Micronaut paired stronger governance with a substantial framework release, and JavaFX, Gradle, and Maven kept pace with the platform.

Just as interesting is what started to emerge around those releases: ahead-of-time compiled code in HotSpot's cache, more adaptive ZGC behavior, portable checkpoint-and-restore hooks for Jakarta EE, independently verifiable JDK binaries, and Java tooling built with both human developers and coding agents in mind.

If September was about shipping Java 27, the next few months will be about seeing which of those changes teams adopt immediately—and which JDK 28 ideas are ready to become the next platform defaults.

## Sources

This edition is based on material published during September 2026 by OpenJDK and Oracle, Eclipse Adoptium and the Eclipse Foundation, Jakarta EE projects, Spring, Quarkus, Micronaut, WildFly, Payara, Gluon/OpenJFX, Apache Maven and Groovy, Gradle, TornadoVM, and official Java community event sites.