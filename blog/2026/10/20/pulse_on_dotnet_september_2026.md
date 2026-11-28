# Pulse on .NET – September 2026

*Your AI-generated monthly roundup of .NET platform updates and community highlights.*

September marked an important transition for the .NET ecosystem. .NET 11 moved into its release-candidate phase, C# 15’s unions and closed hierarchies became substantially more concrete, and the broader platform shifted from preview experimentation toward stabilization ahead of the November release.

The month was not limited to the core platform. ASP.NET Core and Blazor expanded their work around agentic applications, Aspire delivered one of its most substantial dashboard updates, MSTest and NUnit both advanced the testing story, and Visual Studio, Rider, ReSharper, Uno Platform, and Avalonia all shipped meaningful developer-tooling improvements.

For application and library teams, the practical work is becoming clearer. RC1 is the point to validate compatibility in earnest, .NET 8 and .NET 9 are approaching their shared November support deadline, and changes to testing, package verification, and development tooling deserve attention before .NET 11 reaches general availability.

More broadly, September showed several threads converging. C# language features are reaching the web and serialization stacks, agent protocols are connecting to application-specific UI, Native AOT is influencing test infrastructure, and Aspire is expanding from orchestration into a more durable development environment.

Let’s take a look at what happened across the .NET ecosystem during September 2026.

## September at a Glance

| Category | September 2026 highlights |
| --- | --- |
| **Platform** | [.NET 11 RC1](https://devblogs.microsoft.com/dotnet/dotnet-11-rc-1/) introduced the go-live milestone and continued refining the runtime, SDK, MSBuild, NuGet, ASP.NET Core, MAUI, and Windows Forms. |
| **Languages and performance** | C# 15 unions and closed hierarchies reached the ASP.NET Core and JSON stacks, F# added record and delegate improvements, and the annual [.NET 11 performance review](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-11/) detailed another broad set of runtime and library gains. |
| **Web and AI** | Experimental Blazor AI components and the new [.NET SDK for AG-UI](https://devblogs.microsoft.com/dotnet/ag-ui-dotnet-sdk/) created a more coherent foundation for agent-driven application experiences. |
| **Cloud native** | [Aspire 13.6](https://aspire.dev/whats-new/aspire-13-6/) added persistent telemetry and run history, integrated terminals, first-party Java and Rust hosting, and a Native AOT dashboard. |
| **Testing** | MSTest 4.4 expanded Native AOT testing through source generation, while [NUnit 5](https://docs.nunit.org/articles/nunit/release-notes/framework.html) improved type safety and introduced several breaking API changes. |
| **Developer tooling** | Visual Studio 2026 18.10 expanded model-provider, Git, debugging, and NuGet capabilities, while Rider and ReSharper 2026.2.2 improved agent integration and test coverage. |
| **Ecosystem and operations** | Uno Platform 6.7, Avalonia for Visual Studio 12.3, Azure SDK updates, September servicing releases, and Microsoft's NuGet signing-certificate rotation rounded out the month. |

## .NET 11 Release Candidate 1

The headline release of the month was [.NET 11 Release Candidate 1](https://devblogs.microsoft.com/dotnet/dotnet-11-rc-1/), published on September 8. RC1 carries a go-live support license, making it the first .NET 11 build intended for supported production evaluation rather than preview-only experimentation.

The release included updates across the runtime, libraries, SDK, MSBuild, NuGet, C#, F#, ASP.NET Core, Blazor, .NET MAUI, and Windows Forms.

The runtime work included in-process crash reporting on Unix and support for using FP16 hardware instructions for `Half` operations. The libraries added process signaling and termination-status inspection, caller-driven TLS sessions, DNS record resolution on Linux, JSON support for new numeric types and binary schemas, closed-type polymorphism and union serialization, asynchronous options validation, AES Key Wrap, and reusable compression encoders and decoders.

The SDK continued refining several developer workflows. `dotnet test` gained additional support for mobile application testing, run-level controls, and result layouts. Container publishing became more reproducible and can avoid redundant uploads. File-based programs gained Native AOT reuse and formatting support, while `dotnet format` tightened how it discovers configuration files.

MSBuild added APIs for creating and extracting tar archives and for exposing item glob patterns to build tooling. NuGet improved package generation by reusing existing project evaluations and made vulnerable-package updates work more cleanly when warnings are treated as errors.

Together, these changes made RC1 feel less like another feature preview and more like a release that development teams can begin validating seriously.

## C# 15 Unions and Closed Hierarchies

One of the most visible language developments in .NET 11 is C# 15’s support for unions and closed class hierarchies. RC1 stabilized the core union work, including declaration syntax, conversions, pattern matching, exhaustiveness analysis, and integration with `System.Text.Json`.

Microsoft followed the RC1 release with a detailed look at how to [use C# unions and closed hierarchies in ASP.NET Core](https://devblogs.microsoft.com/dotnet/unions-and-closed-hierarchies-in-aspnetcore/).

A union represents a value drawn from a fixed set of case types. Those types do not have to share a base class and can include primitives, classes, interfaces, and nullable types. The compiler understands the complete set of alternatives and can warn when a switch expression does not handle every case.

Closed hierarchies address a related but different scenario. A `closed` base class restricts direct derivation outside its assembly, allowing the compiler to treat the known derived types as a complete family. This preserves normal inheritance and shared behavior while enabling the same kind of exhaustiveness checking associated with unions.

The distinction matters for API design. A union is well suited to an existing discriminator-free contract or to alternatives that cannot share a base class. A closed hierarchy is generally better when the alternatives form a related domain model that an application controls.

The ASP.NET Core integration reaches further than basic JSON serialization. Unions work with Minimal API and MVC request and response bodies, SignalR’s JSON protocol, Blazor JavaScript interop, persisted component state, prerendered component parameters, and OpenAPI generation. OpenAPI describes union cases through `anyOf` schemas, while `System.Text.Json` can serialize the active case without introducing a wrapper object.

September was therefore the month when unions stopped looking like an isolated language feature and started looking like a full-stack .NET capability.

## F# Improvements Continue

F# also received a substantial set of improvements in .NET 11 RC1.

Record spreads make it easier to create a new record from an existing value while changing selected fields. Record constructors offer a more direct construction model, and direct delegate construction reduces ceremony when interoperating with delegate-based APIs.

The compiler also added more efficient interpolated strings and reflection-free formatting for records and discriminated unions. These changes fit the broader direction of .NET 11: reducing runtime reflection, improving trimming and Native AOT compatibility, and making common language patterns more efficient without changing their essential character.

## Performance Improvements in .NET 11

September also brought Stephen Toub’s annual deep dive into the [performance improvements in .NET 11](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-11/).

As in previous releases, the story is not built around one benchmark or one dramatic optimization. It is the accumulation of hundreds of changes across the JIT, runtime, garbage collector, libraries, networking stack, cryptography, text processing, collections, serialization, reflection, threading, and application frameworks.

The improvements include removed bounds checks, fewer allocations, reduced locking, loop optimizations, better constant folding, instruction fusion, more effective SIMD use, fewer system calls, and improved handling of common collection and array operations.

That cumulative approach remains one of .NET’s strengths. Individual changes may be small, but they compound across application startup, request processing, serialization, asynchronous work, collection handling, networking, and sustained service workloads.

## ASP.NET Core and Blazor

ASP.NET Core RC1 finalized the SignalR authentication-refresh APIs, including TypeScript client support and updates to Blazor Server circuits after authentication changes. OpenAPI generation now reflects obsolete APIs and can select a specific environment during build-time document generation. Negotiate authentication gained TLS channel binding, and the release also included experimental Device Bound Session Credentials support.

The most forward-looking addition was a set of experimental Blazor AI components for agentic user interfaces.

Microsoft expanded on that work later in the month with [Build Agentic UI with the new Blazor AI components](https://devblogs.microsoft.com/dotnet/build-agentic-ui-blazor/). The components provide an observable state model and UI building blocks for streamed agent interactions, tool calls, approvals, shared state, predictive state, and generative experiences.

`ChatPage` provides a complete conversational entry point, while lower-level components such as `AgentBoundary`, `MessageList`, `MessageInput`, and `BlockRenderer` support application-specific layouts. `UIAgent` connects the component model to `IChatClient` from `Microsoft.Extensions.AI`, allowing the same UI model to work with local, remote, or provider-specific agents.

The important idea is that an agent interaction does not have to remain a stream of text in a chat box. Tool calls can become application-specific Razor components. Consequential actions can pause for explicit user approval. Agent-generated state can drive ordinary application UI, while predictive changes can remain separate until the user accepts them.

## AG-UI Gets a First-Class .NET SDK

The Blazor work was accompanied by the release of a [first-class .NET SDK for AG-UI](https://devblogs.microsoft.com/dotnet/ag-ui-dotnet-sdk/), developed by the .NET team in collaboration with CopilotKit.

AG-UI, or Agent-User Interaction Protocol, standardizes communication between agents and user-facing applications. Instead of every agent framework exposing a different streaming format, AG-UI represents the agent lifecycle through typed events for messages, tools, state changes, reasoning, interrupts, and run status.

The .NET SDK supports both ends of the connection. `AGUI.Server` can expose an `IChatClient` as an AG-UI endpoint, while `AGUI.Client` lets a .NET application consume an AG-UI agent through `AGUIChatClient`. Because `AGUIChatClient` also implements `IChatClient`, existing application code can use a remote AG-UI endpoint without adopting a separate abstraction.

The SDK lives in the AG-UI repository alongside the TypeScript and Python implementations and is published on NuGet under the MIT license. Microsoft Agent Framework’s AG-UI support for .NET now builds on these shared packages.

Combined with the Blazor AI components, the release created a more coherent end-to-end story: Microsoft.Extensions.AI supplies the abstractions, agent frameworks perform the work, AG-UI carries the interaction, and Blazor renders it as application-specific UI.

## Aspire 13.6

[Aspire 13.6](https://aspire.dev/whats-new/aspire-13-6/) arrived on September 29 with one of the project’s most requested features: persistent dashboard telemetry and run history.

The Aspire dashboard now stores resource snapshots and telemetry in SQLite. Developers can move between the current run and completed runs, inspect earlier resources, logs, traces, and metrics, and pin runs that should not be removed from the rolling history.

The dashboard also gained AppHost-owned terminals and docked REPL experiences for database and cache integrations. PostgreSQL, MySQL, MongoDB, SQL Server, Redis, and Valkey resources can expose authenticated command-line clients directly inside the dashboard.

Aspire 13.6 expanded beyond .NET with first-party Java and Rust hosting. The release also introduced preview Azure Connector Namespace support and Azure Container Apps Sandboxes as an experimental deployment target.

Other updates included coordinated .NET project builds, portable volume paths, TypeScript AppHost configuration through `appsettings.json`, new CLI workflows, improved Visual Studio Code integration, Blazor WebAssembly debugging, Deno hosting, MongoDB replica sets, Microsoft Foundry Toolboxes, and support for remote Foundry Local services.

The dashboard now ships as a Native AOT executable with a refreshed Fluent UI interface. Teams upgrading should review the documented breaking changes, particularly around MongoDB TLS, the Cosmos DB emulator, Azure Front Door origin naming, connection-string environment-variable names, and experimental terminal APIs.

## Testing: MSTest and NUnit 5

September was an unusually significant month for .NET testing.

Microsoft published [Test what you ship: MSTest and Native AOT](https://devblogs.microsoft.com/dotnet/mstest-source-generation/), detailing the Native AOT support available with MSTest 4.4.

MSTest can now publish and run test projects as Native AOT executables. Source generation records test classes, attributes, constructors, and invocation delegates before trimming occurs, allowing tests to remain discoverable without requiring developers to rewrite their normal `[TestClass]` and `[TestMethod]` code.

The goal is not to replace managed test runs. Managed tests remain the fast feedback path, while a targeted Native AOT lane can detect failures that appear only after trimming, ahead-of-time compilation, or removal of unrestricted reflection.

Later in the month, [NUnit 5.0.0](https://docs.nunit.org/articles/nunit/release-notes/framework.html) shipped on September 27. The major release continued NUnit’s move toward stronger type safety and resolved 48 issues.

New capabilities include test and fixture dependencies, `TestContext.ActiveTests`, type-safe expected results for generic `TestCaseData`, warning thresholds for `MaxTime`, and threshold-based success or failure for repeated test cases.

The release also contains breaking changes. APIs based on NUnit-specific test delegates have moved to standard `Action` and `Func` types. `Assert.ThrowsAsync`, `Assert.CatchAsync`, and `Assert.DoesNotThrowAsync` now return tasks and must be awaited. NUnit 5 also dropped .NET 6 as a framework target and changed the interpretation of several platform identifiers to align with modern .NET terminology.

## Visual Studio and JetBrains Tooling

The [Visual Studio 2026 September update](https://learn.microsoft.com/en-us/visualstudio/releases/2026/release-notes#september-update-18100), version 18.10, shipped on September 8.

Bring Your Own Key support entered preview, enabling developers to connect Agent Mode to Microsoft Foundry, OpenAI, Anthropic, Ollama, and supported custom endpoints. The Git agent can explore pull requests in chat and provide navigable links to comments and changed files.

Debugging gained inline highlighting that identifies which operand determined the result of a compound C# condition. Attach to Process added Podman container discovery for .NET and C++ applications. NuGet Audit gained separately configurable audit sources, and Razor Hot Reload received performance and reliability improvements through tighter Roslyn integration.

JetBrains released [Rider and ReSharper 2026.2.2](https://blog.jetbrains.com/dotnet/2026/09/16/rd-rs-2026-2-2/) on September 16.

Rider added an AI Agent Setup widget that brings MCP tools, skills, and hooks into one discoverable entry point. Agent hooks now return more targeted inspection information and report formatting changes as diffs. Rider’s dotCover integration also added coverage support for tests written with TUnit.

JetBrains additionally described its investigation into [Rider and ReSharper startup performance](https://blog.jetbrains.com/dotnet/2026/09/09/why-rider-and-resharper-were-slow-to-start-and-how-microsoft-helped-fix-the-problem/), including the effect of Microsoft Defender scanning on ReSharper’s out-of-process architecture and the collaboration between JetBrains and Microsoft to diagnose the issue.

## Cross-Platform UI

.NET MAUI’s September work arrived as part of .NET 11 RC1, with updates across testing, controls, platform capabilities, XAML, Android, and Apple platforms. The release continued the broader effort to harden the CoreCLR-based mobile runtime path and prepare the platform for .NET 11.

[Uno Platform 6.7](https://platform.uno/blog/uno-platform-6-7/) shipped on September 24. It expanded Android support to Android TV, Android Auto, and Wear OS and introduced semantic styles that allow the same style aliases to work across Material and Simple themes.

Uno’s Skia renderers now repaint only damaged regions rather than the entire surface. The release also added system-aware text scaling, WebAssembly screen-reader improvements, a dedicated Win32 render thread, GPU-accelerated `RenderTargetBitmap`, and WebAssembly support for `GLCanvasElement`.

[Avalonia for Visual Studio 12.3](https://avaloniaui.net/whats-new/12-3) shipped on September 13. Its new previewer gained form-factor selection for common screen sizes and moved out of experimental status, while the legacy previewer became deprecated. The extension also added more AXAML analysis, markup validation, collection and data-type parsing improvements, new editor themes, and image previews for `avares` paths.

## Azure SDK, Servicing, and NuGet Security

The [September Azure SDK release](https://devblogs.microsoft.com/azure-sdk/azure-sdk-release-september-2026/) included new stable and preview packages across the .NET ecosystem.

The .NET PostgreSQL Authentication extension for Npgsql reached version 1.0.0, providing Microsoft Entra ID authentication support. Azure AI Search 12.1.0-beta.2 added support for the `2026-08-01-preview` service API, while several resource-management and provisioning libraries reached their initial stable or beta releases.

Microsoft also published the [.NET and .NET Framework September 2026 servicing updates](https://devblogs.microsoft.com/dotnet/dotnet-and-dotnet-framework-september-2026-servicing-updates/) on September 8. The release delivered .NET 10.0.12, .NET 9.0.20, and .NET 8.0.31, together with security and non-security fixes for supported .NET and .NET Framework versions.

NuGet package consumers received a separate operational notice when Microsoft began [rotating its NuGet author-signing certificate](https://devblogs.microsoft.com/dotnet/microsoft-author-signing-certificate-update-2026/) on September 23.

Most developers are unaffected, but organizations that enforce trusted-signer allow lists or verify Microsoft packages by certificate fingerprint need to add the new certificate. Existing packages retain their original signatures, while newly signed packages transition to the new certificate.

## .NET Conf and the Community

The [.NET Conf 2026 Community Days call for presenters](https://devblogs.microsoft.com/dotnet/dotnet-conf-2026-community-days-call-for-presenters/) opened on September 8.

For 2026, the community is taking a larger role in the final two days of the conference. A community-led team is responsible for selecting talks, hosting the broadcasts, and running Community Days. Proposed sessions can cover web, native, mobile, desktop, AI, IoT, games, containers, microservices, machine learning, data, DevOps, and .NET open-source projects.

That change feels appropriate for a platform whose monthly story increasingly extends beyond any one product team. September combined the core .NET 11 release train with independent framework releases, cross-platform UI work, cloud SDKs, open protocols, community testing projects, and competing development environments.

## Looking Ahead

September made the remaining work before .NET 11 unusually concrete. If you maintain .NET applications, libraries, or development infrastructure, the priorities are now fairly clear:

- Test applications and libraries against .NET 11 RC1, including build, packaging, trimming, and Native AOT scenarios where relevant.
- Plan or complete migrations from .NET 8 and .NET 9 before both releases reach end of support on November 10.
- Review NUnit 5's breaking changes and add a targeted Native AOT test lane where deployed artifacts rely on trimming or ahead-of-time compilation.
- Apply the September servicing updates and rebuild container images against the updated .NET base images.
- Update trusted-signer policies if they pin Microsoft's previous NuGet author-signing certificate.

September was the month .NET 11 started to look finished. RC1 brought the go-live milestone, C# 15’s unions became part of a coherent ASP.NET Core and JSON story, and the annual performance review showed how another year of runtime and library work had accumulated.

At the same time, AG-UI and the Blazor AI components pointed toward a new application model in which agents participate in structured user experiences rather than living behind a chat window.

Aspire 13.6, MSTest 4.4, NUnit 5, Visual Studio 18.10, Rider and ReSharper 2026.2.2, Uno Platform 6.7, and Avalonia for Visual Studio 12.3 made September much broader than a single release-candidate story.

With .NET 11 approaching general availability, the focus now shifts from feature discovery to compatibility testing, production validation, migration planning, and the final work needed to turn the year’s previews into a stable platform release.

See you next month.