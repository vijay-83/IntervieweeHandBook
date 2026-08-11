# Powerful .NET Libraries — CLI (Command-Line) App Development

A curated reference of high-value libraries for building command-line tools and console apps in .NET, grouped by purpose, with what each is used for and where to find official guidance.

---

## 1. App Scaffolding & Hosting

| Library / Framework | Use | Guidance Link |
|---|---|---|
| **System.CommandLine** | Microsoft's official library for parsing command-line args, subcommands, options, and generating help text | https://learn.microsoft.com/en-us/dotnet/standard/commandline/ |
| **.NET Generic Host (Microsoft.Extensions.Hosting)** | Wires up DI, configuration, and logging inside a CLI app, same as ASP.NET Core's hosting model | https://learn.microsoft.com/en-us/dotnet/core/extensions/generic-host |
| **`dotnet tool` (global/local tools)** | Packaging and distributing a .NET CLI app as an installable `dotnet tool` via NuGet | https://learn.microsoft.com/en-us/dotnet/core/tools/global-tools |
| **Native AOT (dotnet publish)** | Compiles the CLI to a single, fast-starting native executable — ideal for CLI tools (no JIT warm-up) | https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/ |

---

## 2. Argument Parsing & Command Frameworks

| Library / Framework | Use | Guidance Link |
|---|---|---|
| **System.CommandLine** | The standard, official parser — supports options, arguments, subcommands, validation, and shell completion | https://learn.microsoft.com/en-us/dotnet/standard/commandline/ |
| **CommandLineParser (commandlineparser/commandline)** | Popular community attribute-based parser (`[Option]`/`[Verb]`) — simple, mature, widely used | https://github.com/commandlineparser/commandline |
| **CliFx** | Framework for building strongly-typed, testable CLI apps with routing between multiple commands | https://github.com/Tyrrrz/CliFx |
| **McMaster.Extensions.CommandLineUtils** | Attribute-driven CLI framework with subcommands, help generation, and response files | https://github.com/natemcmaster/CommandLineUtils |
| **Cocona** | Minimal-API-style CLI framework — build CLIs the same way you build ASP.NET Core Minimal APIs | https://github.com/mayuki/Cocona |

---

## 3. Terminal UI, Output & Interactivity

| Library / Framework | Use | Guidance Link |
|---|---|---|
| **Spectre.Console** | The most popular library for rich terminal UI — tables, trees, progress bars, live displays, markup-styled text, prompts | https://spectreconsole.net/ |
| **Spectre.Console.Cli** | Command routing/DI add-on for Spectre.Console — turns it into a full CLI framework | https://spectreconsole.net/cli/ |
| **Terminal.Gui (gui.cs)** | Full text-based UI (TUI) toolkit — windows, menus, dialogs for building console "apps" with layouts | https://github.com/gui-cs/Terminal.Gui |
| **Colorful.Console** | Adds true-color (24-bit) text and gradient/ASCII-art output to the console | https://github.com/tomakita/Colorful.Console |
| **ShellProgressBar** | Lightweight progress bar rendering for long-running console operations | https://github.com/Mpdreamz/shellprogressbar |
| **Sharprompt** | Interactive prompt library — select lists, confirmations, multi-select, input validation | https://github.com/shibayan/Sharprompt |
| **Figgle** | Renders ASCII-art "FIGlet" banner text for CLI headers/branding | https://github.com/drewnoakes/figgle |

---

## 4. Configuration, Logging & Diagnostics

| Library / Framework | Use | Guidance Link |
|---|---|---|
| **Microsoft.Extensions.Configuration** | Layered config from appsettings.json, env vars, command-line args, user secrets | https://learn.microsoft.com/en-us/dotnet/core/extensions/configuration |
| **Serilog (Console/File sinks)** | Structured logging output to console and files, same as in web apps | https://serilog.net/ |
| **Microsoft.Extensions.Logging** | Standard logging abstraction, easily wired into a CLI via the Generic Host | https://learn.microsoft.com/en-us/dotnet/core/extensions/logging |
| **Polly** | Retry/circuit-breaker resilience for CLI tools calling external APIs/services | https://www.pollydocs.org/ |

---

## 5. File System, Process & OS Interaction

| Library / Framework | Use | Guidance Link |
|---|---|---|
| **System.IO.Abstractions** | Testable file-system abstraction (`IFileSystem`) so CLI file operations can be unit tested | https://github.com/TestableIO/System.IO.Abstractions |
| **CliWrap** | Fluent wrapper for running and piping external processes/command-line tools from .NET | https://github.com/Tyrrrz/CliWrap |
| **Glob / Microsoft.Extensions.FileSystemGlobbing** | Glob-pattern file matching for CLI tools that operate on file sets | https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.filesystemglobbing |

---

## 6. Packaging, Updates & Distribution

| Library / Framework | Use | Guidance Link |
|---|---|---|
| **NuGet (dotnet tool packaging)** | Standard way to package and publish a .NET CLI tool for `dotnet tool install` | https://learn.microsoft.com/en-us/dotnet/core/tools/global-tools-how-to-create |
| **Velopack** | Cross-platform auto-update and installer framework for .NET apps (including CLI tools) | https://docs.velopack.io/ |
| **Squirrel.Windows** | Auto-update framework specifically for Windows desktop/CLI apps | https://github.com/Squirrel/Squirrel.Windows |

---

## 7. Testing

| Library / Framework | Use | Guidance Link |
|---|---|---|
| **xUnit / NUnit** | Standard unit testing frameworks, same as for web apps | https://xunit.net/ · https://nunit.org/ |
| **CliFx.Testing / Spectre.Console.Testing** | Test harnesses to simulate CLI input/output and assert console rendering | https://github.com/Tyrrrz/CliFx · https://spectreconsole.net/testing |
| **Verify** | Snapshot testing library — great for asserting CLI console output stays stable across changes | https://github.com/VerifyTests/Verify |

---

## Notes

- **For a new CLI project**, a strong modern default stack is: **System.CommandLine** (parsing) + **Spectre.Console** (rich output/prompts) + **Generic Host** (DI/config/logging) + **Native AOT** publish for fast startup and single-file distribution.
- If you want an all-in-one framework rather than assembling pieces, **Spectre.Console.Cli** or **CliFx** give you parsing + DI + testability out of the box.
- Package and ship your tool as a **`dotnet tool`** via NuGet — this is the standard, cross-platform way .NET developers install CLI utilities (`dotnet tool install -g yourtool`).
- Use **Native AOT** for CLI tools where startup latency matters — a JIT-compiled console app has a noticeably slower cold start than a Native AOT binary.
