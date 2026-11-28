# Pulse on .NET – June 2026 Edition

*Your AI-generated monthly roundup of .NET platform updates and community highlights.*

Welcome to the June 2026 edition of Pulse on .NET, where we recap the latest updates from Microsoft’s official releases and the broader .NET ecosystem over the past month. June was a busy midpoint in the .NET 11 release cycle, featuring **.NET 11 Preview 5**, regular servicing updates, major tooling evolution around AI workflows, and strong ecosystem momentum across Azure, MAUI, and cross-platform development.

As with previous editions, we’ll cover:
- Core releases and servicing updates  
- Tooling evolution (IDEs, AI, build systems)  
- Ecosystem developments (libraries, SDKs, frameworks)  
- Community highlights (blogs, videos, events)  

Let’s dive in.

---

## Key Updates in June 2026 by Category

| Category | Update | Release Date | Highlights |
|----------|--------|--------------|------------|
| Releases | .NET 11 Preview 5 | June 9, 2026 | Broad improvements across runtime, SDK, C#, ASP.NET Core, MAUI, EF Core https://devblogs.microsoft.com/dotnet/dotnet-11-preview-5/ |
| Releases | .NET 10.0.9 / 9.0.17 / 8.0.28 | June 9, 2026 | Security + reliability fixes (3 CVEs) https://devblogs.microsoft.com/dotnet/dotnet-and-dotnet-framework-june-2026-servicing-updates/ |
| Releases | SkiaSharp 4.0 | June 29, 2026 | First stable v4 release https://devblogs.microsoft.com/dotnet/skiasharp-4-0-stable/ |
| Tooling | Visual Studio 2026 (18.7.x) | June 2026 | IDE updates + Copilot fixes https://learn.microsoft.com/en-us/visualstudio/releases/2026/release-notes |
| Tooling | MSBuild MCP Server | June 17, 2026 | AI build diagnostics https://devblogs.microsoft.com/dotnet/msbuild-binlog-mcp-server/ |
| Tooling | MCP in CI | June 30, 2026 | AI diagnostics in pipelines https://devblogs.microsoft.com/dotnet/mcp-build-diagnostics-workflows/ |
| Tooling | Rider 2026.2 EAP | June 2026 | AI agent validation https://blog.jetbrains.com/dotnet/2026/06/08/rider-2026-2-code-quality-check-hooks-for-ai-agents/ |
| Ecosystem | Azure SDK (.NET) | June 2026 | 112 packages released https://azure.github.io/azure-sdk/releases/2026-06/dotnet.html |
| Community | dotInsights | June 12, 2026 | Community roundup https://blog.jetbrains.com/dotnet/2026/06/12/dotinsights-june-2026/ |
| Community | .NET Day | June 2026 | AI modernization event https://aka.ms/dotnetday/playlist |

---

## Official Releases Roundup: .NET 11 Preview 5 and Servicing Updates

The biggest release in June was **.NET 11 Preview 5**, published on June 9.

https://devblogs.microsoft.com/dotnet/dotnet-11-preview-5/

This preview continues the trend seen in earlier .NET 11 previews: incremental but meaningful improvements across the entire stack rather than a single headline feature.

### Platform Highlights

#### Libraries
- JSON Lines serialization support in `System.Text.Json`
- LINQ now supports **full outer joins**
- X25519 key agreement added to cryptography APIs
- Continued improvements in networking and diagnostics APIs

#### Runtime
- Faster async suspension and resumption
- JIT optimizations targeting real-world workloads
- Garbage collector trimming and compaction improvements

#### SDK
- File-based apps can now reference additional files
- Build-time **vulnerability and end-of-life checks**
- New MCP Server template (`dotnet new mcpserver`)

#### C#
- **Union types (preview)**
- Closed class hierarchies
- Continued “unsafe evolution” work

#### ASP.NET Core
- Blazor SSR client-side validation
- QuickGrid improvements
- Better WebAssembly tooling

#### .NET MAUI
- Large reliability update wave
- Android API 37 stabilization
- Azure Maps-backed Windows Maps support

#### Entity Framework Core
- File-based app support for `dotnet ef`
- SQL Server 2022 as default compatibility
- Cleaner SQL query generation

👉 Download: https://dotnet.microsoft.com/en-us/download/dotnet/11.0

---

### Servicing Releases (Patch Tuesday)

https://devblogs.microsoft.com/dotnet/dotnet-and-dotnet-framework-june-2026-servicing-updates/

Microsoft released:
- .NET 10.0.9  
- .NET 9.0.17  
- .NET 8.0.28  

These updates:
- Fixed **three CVEs**  
- Delivered reliability improvements  
- Maintained platform stability  

👉 GitHub tracking: https://github.com/dotnet/core/issues/10443

Notably:
- No .NET Framework updates this month  
- Strong signal that .NET 10 remains stable post-launch  

---

### SkiaSharp 4.0 Goes Stable

https://devblogs.microsoft.com/dotnet/skiasharp-4-0-stable/

SkiaSharp 4.148.0 marks a **major milestone** for cross-platform graphics in .NET.

Key improvements:
- Updated Skia engine (m148)
- Variable font axis control
- Animated WebP encoding
- Major API cleanup
- Improved memory lifecycle management

Impact:
- .NET MAUI apps
- Uno Platform apps
- Cross-platform UI rendering stacks

This is one of the most important ecosystem releases of the month.

---

## Tooling Updates: AI Workflows Take Center Stage

### Visual Studio 2026 (June Update)

https://learn.microsoft.com/en-us/visualstudio/releases/2026/release-notes

The June update (18.7.x) focused on:
- Copilot stability fixes
- IDE responsiveness improvements
- Theme customization enhancements

Multiple patch releases followed quickly:
- 18.7.1
- 18.7.2
- 18.7.3

This reflects ongoing refinement of **AI-first IDE experiences**.

---

### AI-Powered Build Diagnostics (MCP)

https://devblogs.microsoft.com/dotnet/msbuild-binlog-mcp-server/  
https://devblogs.microsoft.com/dotnet/mcp-build-diagnostics-workflows/

Microsoft introduced:
- **Binlog MCP Server**
- AI-powered MSBuild diagnostics
- CI/CD integration

Capabilities:
- Analyze build failures
- Identify performance bottlenecks
- Provide structured AI insights

This represents a shift toward:
➡️ AI-native debugging  
➡️ Automated diagnostics pipelines  

---

### JetBrains Rider 2026.2 EAP

https://blog.jetbrains.com/dotnet/2026/06/08/rider-2026-2-code-quality-check-hooks-for-ai-agents/

Highlights:
- AI agent validation hooks
- WPF Hot Reload
- Improved NuGet tooling
- Performance improvements

Key innovation:
➡️ IDE-enforced validation for AI-generated code  

---

## Ecosystem News: Azure and Cloud-Native Development

### Azure SDK for .NET

https://azure.github.io/azure-sdk/releases/2026-06/dotnet.html

June delivered:
- **112 packages**
- 24 stable releases
- 43 beta packages

Focus areas:
- AI services
- Storage
- Identity
- Cloud resource management

---

### Azure Functions + AI Tooling

https://devblogs.microsoft.com/azure-sdk/

Key developments:
- MCP integration
- AI-powered developer workflows
- Improved diagnostics and deployment

This reinforces Azure’s move toward:
➡️ AI-native cloud development  

---

## Community Highlights

### JetBrains dotInsights

https://blog.jetbrains.com/dotnet/2026/06/12/dotinsights-june-2026/

Topics included:
- EF Core performance improvements
- MAUI integrations
- Identity (SAML, passkeys)
- Observability (OpenTelemetry)
- Cryptography trends

---

### .NET Day on Agentic Modernization

https://aka.ms/dotnetday/playlist

Highlights:
- AI-driven modernization workflows
- MCP-based development patterns
- Real-world enterprise scenarios

---

## Closing Thoughts

June 2026 showed a clear shift in the .NET ecosystem:

- .NET 11 is stabilizing rapidly  
- AI is becoming part of everyday development workflows  
- Tooling is evolving around agentic development  
- The ecosystem remains highly active  

The key takeaway:

➡️ .NET is not just evolving as a platform—it is evolving as an **AI-augmented developer experience**.