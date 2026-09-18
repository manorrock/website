# Pulse on .NET – August 2026

August was a busy month for .NET. We got the final preview of .NET 11 before the release-candidate phase, a substantial security servicing release across supported .NET versions, changes to the way NuGet packages are published, and another round of improvements to the increasingly agent-oriented developer experience in Visual Studio and Visual Studio Code.

Microsoft and the broader ecosystem were active as well. Aspire continued its rapid release cadence, xUnit.net shipped a major release with Native AOT support, Orleans delivered another substantial update, the official Model Context Protocol C# SDK continued evolving, and the .NET community had an important conversation about sustainable open-source maintenance.

August was not simply another month of release churn. It exposed concrete work for application teams, package publishers, open-source consumers, and anyone planning a move from .NET 8 or .NET 9 before their shared November support deadline.

More broadly, several long-running trends started to converge. Native AOT reached more of the toolchain. C# language work began influencing framework design. Testing and source-control tooling became better suited to automated development workflows. MCP appeared in both AI frameworks and ordinary operations tools. And the less fashionable concerns—security servicing, support lifecycles, dependency governance, and publishing credentials—remained just as important.

Let's take a look at what happened in the .NET ecosystem during August 2026.

## August at a Glance

| Category | August 2026 highlights |
| --- | --- |
| **Platform** | [.NET 11 Preview 7](https://devblogs.microsoft.com/dotnet/dotnet-11-preview-7/) enabled the Native AOT `dotnet` CLI and MSBuild server by default, expanded `dotnet test`, advanced C# 15, and improved ASP.NET Core, MAUI, EF Core, and Windows Forms. |
| **Security and support** | The [August servicing updates](https://devblogs.microsoft.com/dotnet/dotnet-and-dotnet-framework-august-2026-servicing-updates/) addressed ten CVEs. .NET 8 and .NET 9 both approach end of support on November 10. |
| **Supply chain** | NuGet.org [reduced API-key lifetimes](https://devblogs.microsoft.com/dotnet/strengthening-nuget-supply-chain-security-reducing-api-key-lifetime/) to 30 days and recommended Trusted Publishing with OpenID Connect. |
| **Developer tooling** | [Visual Studio 2026 18.9](https://learn.microsoft.com/en-us/visualstudio/releases/2026/release-notes#august-update-1890) added agent controls, local code review, Git worktrees, richer diffs, and expanded pull-request tooling. Visual Studio Code [1.134](https://code.visualstudio.com/updates/v1_134) and [1.135](https://code.visualstudio.com/updates/v1_135) continued investing in multi-session agent workflows. |
| **Testing** | [Microsoft.Testing.Platform 2.3](https://devblogs.microsoft.com/dotnet/microsoft-testing-platform-reporting/) improved CI reporting and crash resilience, while [xUnit.net v3 4.0](https://xunit.net/releases/v3/4.0.0) added Native AOT test builds and broader parallelization. |
| **Cloud native and AI** | [Aspire 13.5](https://github.com/microsoft/aspire/releases/tag/v13.5.0), the [MCP C# SDK 2.x](https://github.com/modelcontextprotocol/csharp-sdk/releases), and experimental Microsoft.Extensions.AI routing and failover expanded the infrastructure available to agent-oriented applications. |
| **Ecosystem and community** | [Orleans 10.3](https://github.com/dotnet/orleans/releases/tag/v10.3.0), CritterWatch 1.0, the [.NET Foundation's maintenance-fee guidance](https://dotnetfoundation.org/news-events/detail/.net-foundation-statement-on-open-source-maintenance-fees), and the [.NET Conf 2026 announcement](https://devblogs.microsoft.com/dotnet/dotnet-conf-2026/) rounded out the month. |

## .NET 11 Reaches Its Final Preview

Microsoft released .NET 11 Preview 7 on August 11. It was the final preview before .NET 11 moved into its release-candidate phase with RC1 on September 8, on the way to general availability in November.

At this point in the release cycle, the shape of .NET 11 is becoming much clearer. No single Preview 7 feature redefines the platform, but the cumulative work shows a release focused on performance, deployment flexibility, safer defaults, and a smoother developer inner loop.

### Runtime, SDK, and C# 15

The runtime and base libraries continued to expand with IEEE 754 decimal floating-point types, generic `Complex<T>`, HTTP request compression, configurable connection eviction, DNS record-resolution APIs, password-protected ZIP archives, asynchronous `ChangeToken.OnChange`, and additional `System.Text.Json` polymorphism capabilities.

There was also continued work on Native AOT, JIT code generation, async tiering, and WebAssembly. One particularly interesting milestone was getting the libraries test suite running on CoreCLR WebAssembly. The runtime work also included tail-await optimizations and additional Native AOT improvements.

These changes are incremental, but that is increasingly the point. Startup time, throughput, deployment size, and support for constrained environments have improved over several release cycles rather than through one dramatic rewrite.

The SDK received changes that may be more immediately noticeable. The Native AOT implementation of the `dotnet` CLI is now enabled by default, as is the MSBuild server. Reusing an MSBuild process between command-line invocations should reduce startup overhead in workflows that repeatedly invoke `dotnet build` or other MSBuild-backed commands.

`dotnet test` gained run-level `--timeout`, `--maximum-failed-tests`, support for `Microsoft.Build.Traversal` projects, improved reporters, and additional .NET MAUI device and environment selection. File-based applications gained `dotnet reference`, while tool packaging added support for custom runtime-identifier matrices.

For teams maintaining frameworks, libraries, build infrastructure, or developer tooling, Preview 7 is the point where serious .NET 11 compatibility testing should already be underway. It is worth testing more than API compatibility. Changes to MSBuild process reuse, Native AOT tooling, and test infrastructure can affect CI isolation, packaging, and build behavior even when application source continues to compile.

C# 15 continued to take shape alongside the platform. The biggest language feature remains union types, which describe a closed set of possible types and integrate that information with conversions, pattern matching, exhaustiveness analysis, and nullability flow analysis.

Preview 7 improved union pattern matching so that patterns can work with either the union or its contained value. Exhaustiveness analysis was also extended to type parameters constrained to closed types.

Union types sit alongside closed hierarchies, collection-expression arguments, extension indexers, and continued changes to unsafe-code rules. Preview 7 also introduced labeled `break` and `continue`, allowing nested code to explicitly target an outer loop or switch. That provides a structured alternative to some flag variables and `goto` statements without introducing unrestricted non-local jumps.

Union types remain the C# 15 feature I am watching most closely because they are already influencing framework design rather than developing as an isolated compiler feature.

### Web, Client, and Data Development

ASP.NET Core Preview 7 included several practical improvements. Blazor Server circuits can now automatically pause after a period of inactivity, allowing applications to trade some reconnection work for lower server resource consumption when users leave sessions idle.

Blazor also gained a `CacheView` component for caching static server-rendered output. `QuickGrid` added an initial item index and programmatic scrolling, new analyzers identify common Blazor mistakes, and the WebAssembly toolchain moved to Emscripten 6.

Elsewhere, Razor can bind literal attributes to union-typed component parameters, validation localization improved, the SignalR .NET client can refresh authentication after redirects, and OpenAPI 3.2 support expanded to include Server-Sent Events.

The early interaction between union types, Razor, and Minimal API scenarios is encouraging. A language feature usually has a better chance of becoming genuinely useful when the surrounding frameworks find natural applications for it before the language release is complete.

.NET MAUI Preview 7 brought cross-platform passkey authentication and continued work on incremental XAML Hot Reload. It also added Shell route templates, AOT-safe `RelativeSource` bindings, and a formalized path for third parties to provide additional platform backends.

The .NET MAUI Community Toolkit followed its late-July 15.0.0 release with [15.0.1](https://github.com/CommunityToolkit/Maui/releases/tag/15.0.1) on August 21, removing its remaining dependencies on internal MAUI APIs and correcting a Windows App SDK version mismatch. Maintainers recommend using 15.0.1 or later for applications preparing for .NET 11.

Entity Framework Core 11 continued improving query translation, migrations, and the Azure Cosmos DB provider, while SQLite gained support for `Half`. There was no single dominant EF Core feature in August, but provider improvements that eliminate application-side workarounds often have more immediate value than larger architectural announcements.

Windows Forms received new opt-in .NET 11 visual styles, notifications for system visual-setting changes, painting suspension during bulk layout, toggle-switch appearances, and deferred form presentation to reduce white flashes when dark-mode applications start. That last item is a good example of the polish that matters in desktop applications: technically supporting dark mode is not the same as making it feel native.

## Security and Support Deadlines

August was not a routine servicing month. Microsoft released .NET 10.0.11, .NET 9.0.19, and .NET 8.0.30 on August 11, addressing ten vulnerabilities across the supported releases.

The fixes covered remote code execution, information disclosure, elevation of privilege, denial of service, and security bypasses. Updated runtime, ASP.NET Core, Entity Framework Core, container image, installer, and Linux package artifacts were released as part of the servicing cycle.

For organizations running .NET in containers, patching the host is not enough. Applications built on older .NET base images need to be rebuilt and redeployed against the updated images.

The [corresponding .NET Framework cumulative update](https://learn.microsoft.com/en-us/dotnet/framework/release-notes/2026/08-11-august-cumulative-update) addressed additional vulnerabilities across supported .NET Framework versions, but it also introduced WPF printing regressions.

Some applications encountered `System.IO.FileFormatException` while printing or generating PDF or XPS documents using certain fonts, including Calibri. Another issue affected in-memory XPS documents registered in `PackageStore` using a `pack://` identity.

Microsoft documented temporary `AppContext` switches for affected applications, but those switches disable protections introduced by the security update. They should therefore be treated as emergency compatibility measures, not permanent configuration. The original font-related problem was subsequently addressed in the September cumulative update, while Microsoft was still investigating the package-boundary scenario at the time of this review.

If you maintain a WPF application that prints or generates XPS or PDF documents, this was a servicing update worth testing rather than assuming Patch Tuesday would be uneventful. If you used a compatibility switch, make sure it does not quietly remain enabled after the corresponding fix is available.

There is another date worth putting on the calendar: [.NET 8 and .NET 9 both reach end of support](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core) on November 10, 2026.

Applications will not suddenly stop running, but Microsoft will stop providing security fixes, servicing releases, and technical support for those versions. For most production workloads, the obvious destination is .NET 10 LTS, which is supported through November 14, 2028.

The shared deadline is particularly notable because .NET 8 is an LTS release while .NET 9 is an STS release. With November approaching, organizations with significant .NET 8 or .NET 9 deployments should treat migration as current work rather than future planning.

## Publishing, Tooling, and Testing Evolve for Agent Workflows

NuGet.org made an important supply-chain change in August. Beginning August 17, newly created API keys have a maximum lifetime of 30 days. Keys created before that date are scheduled to expire on November 1, 2026.

This matters most for package maintainers with infrequent release pipelines. A publishing secret that has quietly worked for years may fail when the next emergency release needs to go out.

Microsoft's preferred solution is NuGet Trusted Publishing. It uses OpenID Connect to validate information about the CI workload—such as its repository, workflow, and optionally deployment environment—and issues a temporary publishing credential for that operation.

For projects publishing from GitHub Actions or GitLab, Trusted Publishing is considerably more attractive than automating monthly API-key rotation. Publishers using other systems should at least inventory their credentials, tighten package scopes, and ensure expiration notifications reach an actively monitored account.

Visual Studio 2026 version 18.9 also arrived on August 11, with much of the release focused on Copilot and source-control workflows.

Developers can select low, medium, or high thinking effort for supported models and see more information about model capabilities, context-window sizes, and cost. That visibility becomes increasingly important as development environments move from a single default model toward models with different capabilities, latency, and pricing.

Copilot's Git agent can review uncommitted changes or commits locally and present comments inline without requiring a pull request. Organizations can distribute custom agents across repositories.

The Git improvements extend beyond AI. Visual Studio now supports Git worktrees, first-class submodule management, unified multi-file summary diffs, commit-range review, comparisons against the working tree, richer pull-request timelines, and pull-request auto-completion controls.

Git worktrees are particularly relevant in an agent-heavy development environment. They allow multiple independent branches to be checked out simultaneously without maintaining multiple clones. That is useful for humans, but arguably even more useful when several coding agents need isolated workspaces.

The combination of worktrees, local review, richer diffs, and pull-request tooling suggests that source control is becoming part of the contract between developers and agents rather than simply infrastructure underneath the IDE.

Visual Studio Code continued in the same direction through two August releases. Version 1.134 added side-by-side chat groups, a prompt timeline that links prompts to their file changes, full-conversation search, and the option to open local HTML files in the integrated browser by default.

Version 1.135 expanded the Agent Host experience, allowing the same agent session to connect from multiple VS Code windows. It also made recent Copilot or Claude sessions created in other applications available to continue in VS Code, added detailed per-turn token usage, and streamlined the Agents window.

The larger trend is that AI chat is becoming less of a feature embedded in an editor and more of a persistent development object. Once agent sessions span repositories, branches, prompts, and tools, reviewable diffs, explicit context, portable configuration, and durable test evidence become essential.

Testing infrastructure moved in parallel with those IDE changes. Microsoft.Testing.Platform 2.3 improved the connection between test execution and CI systems. GitHub Actions can receive failures as annotations and job summaries, while Azure DevOps can publish results during the run and use pipeline history to help distinguish recurring flaky tests from likely regressions.

TRX output is streamed as tests complete, allowing a crashed host to leave useful partial results instead of losing the entire run. The reporting work also includes multiple formats, stable JSON discovery, and better handling of interrupted runs. Most of these capabilities require Microsoft.Testing.Platform 2.3 or later; the GitHub Actions, JUnit, and CTRF reporters were still distributed as preview packages when Microsoft published the announcement.

These may sound like infrastructure details, but they become increasingly valuable as agents take on more implementation and validation work. Running tests is easy; making the resulting evidence durable, understandable, and reviewable is harder.

xUnit.net v3 4.0 reinforced that direction with Native AOT test builds, full test-case parallelization, Microsoft.Testing.Platform v2 by default, class and method orderers, asynchronous lifecycle hooks, per-test assertion-message formatting, and a cross-platform console runner distributed as a .NET tool.

Official support for Microsoft.Testing.Platform v1 and Mono was removed. Most application test suites should not require dramatic changes, but extension authors and teams with customized infrastructure should review the breaking changes before upgrading.

Native AOT testing is particularly noteworthy because it removes another obstacle for projects that want AOT compatibility to be part of normal development and CI rather than something discovered late in deployment.

## The Cloud-Native and AI Ecosystem Keeps Expanding

Aspire 13.5 arrived on August 18 with a richer dashboard, interactive resource prompts, experimental browser-based terminals, improved persistent-data handling for Kubernetes workloads, support for modeling existing Azure resources, and simpler CLI installation.

What I find interesting about Aspire's recent releases is how much attention is being paid to the complete developer inner loop. The resource model is no longer just about describing and starting an application. Developers increasingly use the same environment to describe resources, start them, interact with them, inspect telemetry, open terminals, and eventually carry that model into deployment.

Aspire increasingly looks less like a collection of local orchestration helpers and more like a development model spanning application composition, observability, interaction, and deployment.

The official MCP C# SDK moved quickly from [version 2.0](https://github.com/modelcontextprotocol/csharp-sdk/releases/tag/v2.0.0) at the end of July to [version 2.1](https://github.com/modelcontextprotocol/csharp-sdk/releases/tag/v2.1.0) and [version 2.2](https://github.com/modelcontextprotocol/csharp-sdk/releases/tag/v2.2.0) during August. Version 2.0 introduced stateless-by-default HTTP, discovery-first negotiation, and stronger OAuth and token-cache safety. Version 2.2 added hybrid stateful and stateless HTTP serving so clients using different protocol revisions can share an endpoint.

MCP is moving from an interesting protocol used by early agent implementations toward infrastructure that .NET developers may encounter whenever applications expose tools, resources, prompts, and other capabilities to AI systems.

[Microsoft.Extensions.AI](https://devblogs.microsoft.com/dotnet/routing-and-failover-for-microsoft-extensions-ai/) also introduced experimental routing and failover primitives for applications using multiple models or endpoints.

That work is less flashy than another chat interface, but it addresses real production concerns. Once an application depends on multiple models, selection, fallback, endpoint health, retries, resilience, and cost become architectural decisions rather than exception-handling details.

Orleans 10.3 was another substantial ecosystem release. It delivered reliability and performance improvements across clustering, streaming, transactions, persistence, code generation, and providers.

The release added or improved support around AWS Kinesis, Firestore, Redis, Azure Table journaling, incremental source generation, OpenTelemetry conventions, and safer JSON type handling. It also included work on durable distributed-directory handoffs and additional journal and transactional-storage providers.

Orleans continues to be one of the more interesting parts of the .NET ecosystem for distributed systems, particularly as actor-style application models intersect with cloud-native hosting and increasingly autonomous workloads.

## Open Source, Community, and Events

One of August's more important community discussions was not about a new API. It was about how open-source software gets maintained.

Projects in the .NET ecosystem, including Polly, have been exploring Open Source Maintenance Fee, or OSMF, models. On August 19, the .NET Foundation published its position.

The Foundation did not endorse or oppose maintenance-fee models. Instead, it emphasized transparency around the terms attached to packages, binaries, hosted services, support, and release channels.

For .NET Foundation projects, source must remain freely available under an approved permissive license. Mandatory dependencies generally must remain permissively licensed, with limited exceptions for platform- or hardware-specific libraries, and a build script must be capable of producing functionally equivalent artifacts apart from code signing.

That leaves room for maintainers to charge for maintainer-produced binaries, services, or other value-added offerings while keeping the underlying project open source.

Regardless of where you stand on the model, dependency governance can no longer mean only checking a package's license identifier and scanning it for vulnerabilities. Organizations increasingly need to understand how consumed artifacts are produced, which terms apply to them, whether public source can reproduce them, which mandatory dependencies they introduce, and what sustainability model supports critical projects.

[CritterWatch 1.0](https://jasperfx.net/news/critterwatch-1-0-is-here.html) provided a smaller but telling example of another ecosystem trend. It shipped as an operations and management console for Critter Stack applications, with projection inspection, dead-letter management, alerting, and other operational tooling.

Its commercially licensed features include an MCP server that allows agents to inspect and act on operational information within configured permissions. MCP is no longer limited to AI-specific frameworks; traditional developer and operations tools are beginning to expose their capabilities to agents as well.

The .NET MAUI Community Toolkit, OpenTelemetry, and other infrastructure projects continued their regular release cadence. Community blogs, videos, standups, and conference recordings covered C# 15 union types, .NET 11 performance, ASP.NET Core security, Blazor, MAUI, Aspire, Orleans, Kubernetes optimization, MCP servers, agent workflows, and application modernization.

Community roundups such as dotInsights and WeekRef.NET remain useful because late in a platform release cycle, official coverage naturally concentrates on the upcoming .NET release while independent libraries continue shipping.

The various .NET Community Standups also continued bridging formal release notes and the people building the platform. They are particularly useful during preview season because they provide design motivation and demonstrations that are difficult to capture in a bullet list.

Microsoft announced that .NET Conf 2026 will take place November 10–12. The free online event will launch .NET 11 and is likely to cover C# 15, runtime and performance improvements, MAUI's continued CoreCLR work, ASP.NET Core and Blazor, OpenAPI 3.2, MCP, AI development, Aspire, and the broader platform.

November 10 is also the end-of-support date for .NET 8 and .NET 9. That makes this year's .NET Conf an unusually clear dividing line between the outgoing and incoming generations of .NET.

JetBrains also announced [.NET Day Online 2026](https://blog.jetbrains.com/dotnet/2026/08/10/jetbrains-dotnet-day-online-2026-save-the-date-and-submit-a-talk/) for October 7. The free one-day event will feature practical talks, live demonstrations, and community speakers, with sessions available afterward. It is a useful reminder that the .NET community extends well beyond Microsoft's conferences and tooling.

## Looking Ahead

August made the remaining work before .NET 11 unusually concrete. If you maintain .NET applications, libraries, or publishing infrastructure, the priorities are now fairly clear:

- Apply the August security updates and rebuild affected container images.
- Test applications and libraries against [.NET 11 RC1](https://devblogs.microsoft.com/dotnet/dotnet-11-rc-1/) and continue monitoring the release-candidate updates.
- Start or finish migrations away from .NET 8 and .NET 9 before November 10.
- Move NuGet publishing pipelines away from long-lived API keys, preferably to Trusted Publishing.
- Review xUnit.net and Microsoft.Testing.Platform changes if you maintain custom test infrastructure.
- Review the licensing, artifact terms, and sustainability model of important open-source dependencies.

The .NET 11 feature set is largely visible now. September has already brought the first release candidate, and there is not much runway left before .NET Conf and general availability in November.

What stands out about August is that the story is no longer simply ".NET 11 is coming." The platform, tooling, and ecosystem work are beginning to converge.

Native AOT is reaching the CLI and testing ecosystem. C# language features are influencing ASP.NET Core and Razor. Testing infrastructure is becoming better suited to automated development workflows. Git worktrees and local review are becoming first-class IDE features just as coding agents need isolated workspaces and reviewable changes. MCP is appearing in both AI frameworks and ordinary developer tools. Aspire is expanding from application composition into a broader development and operations workflow.

At the same time, some decidedly non-AI concerns remain just as important: security servicing, support lifecycles, publishing credentials, dependency governance, and sustainable open-source maintenance.

That combination probably tells us more about where .NET is heading than any individual Preview 7 feature.

See you next month.
