<p align="center">
  <img src="assets/profile-architecture.svg" alt="ntoskrnl7 portfolio architecture map" />
</p>

# Jung-Kwang Lee / ntoskrnl7

I build software at the boundary between low-level systems and application runtimes: Windows kernel drivers, C/C++ runtime layers, Win32 APIs, Chromium/Electron internals, media pipelines, and typed messaging for TypeScript and Rust applications.

The common thread across my projects is not one framework. It is making difficult boundaries explicit: kernel/user mode, native/browser, main/renderer, worker/process, protocol/runtime, old compiler/new language feature, and unsupported upstream behavior/product requirements.

<p>
  <a href="https://github.com/ntoskrnl7/crtsys"><img alt="crtsys stars" src="https://img.shields.io/github/stars/ntoskrnl7/crtsys?style=flat-square&label=crtsys" /></a>
  <a href="https://github.com/ntoskrnl7/win32-ex"><img alt="win32-ex stars" src="https://img.shields.io/github/stars/ntoskrnl7/win32-ex?style=flat-square&label=win32-ex" /></a>
  <a href="https://github.com/ntoskrnl7/ext"><img alt="ext stars" src="https://img.shields.io/github/stars/ntoskrnl7/ext?style=flat-square&label=ext" /></a>
  <a href="https://github.com/ntoskrnl7/electron-port-workspace"><img alt="electron-port-workspace stars" src="https://img.shields.io/github/stars/ntoskrnl7/electron-port-workspace?style=flat-square&label=electron-port-workspace" /></a>
  <a href="https://www.npmjs.com/package/electron-cdp-utils"><img alt="electron-cdp-utils npm" src="https://img.shields.io/npm/v/electron-cdp-utils?style=flat-square&label=electron-cdp-utils" /></a>
  <a href="https://www.npmjs.com/package/typed-message-transport"><img alt="typed-message-transport npm" src="https://img.shields.io/npm/v/typed-message-transport?style=flat-square&label=typed-message-transport" /></a>
</p>

## Portfolio Map

| Layer | What I usually work on | Representative projects |
| --- | --- | --- |
| Kernel runtime | Bringing C/C++ runtime and STL-like development patterns into Windows driver environments | [crtsys], [ldk] |
| Native Windows | Service, process, session, token, privilege, SID, security descriptor, and Win32 helper layers | [win32-ex] |
| C++ utility layer | Small reusable components for process control, result types, URI parsing, units, string handling, versioning, callbacks, and portability | [ext], [util-linux-cpp], [ci-version] |
| Chromium media / Electron ports | Linux VA-API HEVC/H.265 work, Widevine/CDM integration, Electron source patch stacks, and feature ports that expose unavailable runtime capabilities | [electron-port-workspace] |
| Electron runtime APIs | CDP automation, execution-context tracking, worker/frame integration, custom protocol routing, and browser identity control | [electron-cdp], [electron-protocol-provider] |
| Typed messaging | Type-safe request/response contracts across MessagePort, workers, Node.js processes, WebSocket-style transports, and application boundaries | [typed-message-transport], [wsmq-rs], [service-rs] |
| Small tools and bridges | Tiny type utilities, native bridge experiments, build/release helpers | [ts-default], [isim-rs] |

## Selected Work

### crtsys

[crtsys] is a C/C++ runtime library for Windows kernel-driver development. The goal is to make driver code feel closer to normal C++ application development while staying inside the constraints of WDK, kernel stack size, IRQL rules, and driver lifecycle.

It focuses on:

- CRT and Microsoft STL support for kernel drivers without simply vendoring Microsoft CRT/STL sources into the project
- C++ language/runtime features such as initialization, exceptions, threading primitives, futures, mutexes, condition variables, and iostream-style APIs
- `ntl` helpers for driver entry, device objects, NTSTATUS handling, RPC-style user/kernel communication, IRQL helpers, spin locks, resources, and stack expansion
- CMake/CPM-based integration across Visual Studio and Windows Kit versions

This is the project that best represents my systems-runtime work: not only wrapping APIs, but rebuilding enough of the surrounding environment so higher-level C++ code can exist in a restricted runtime.

### electron-port-workspace and Chromium/Electron Feature Ports

[electron-port-workspace] is where I keep reusable Electron and Chromium feature work as portable patch bundles. The point is not just managing patches. It is a way to carry real product capabilities across Electron major versions, upstream `main`, and temporary test branches without losing reproducibility.

Some of the ports are for features that are not available in stock Electron or Chromium builds:

- Linux Chromium/Electron VA-API HEVC/H.265 work, including NVIDIA VA-API native pixmap/modifier import fixes, WebCodecs fallback handling, HEVC Main/Main10/4:4:4 encode support, HEVC output-level plumbing, bitstream builder/parser extensions, stream reset fixes, and Intel iHD-specific stabilization
- Widevine/CDM integration for Electron builds, including Chromium CDM renderer visibility, build defaults, version-aware CDM resolving, manifest validation, host-version checks, and license-acknowledged package assembly
- `session.registerPreloadScript` extensions for frames, subframes, dedicated workers, shared workers, and service workers, including early empty-frame timing fixes and PDF renderer edge cases
- trusted `webContents.dispatchInputEvent()` forwarding for keyboard, mouse, wheel, touch, text insertion, and IME composition with Chromium input ACK visibility
- focused editable text and caret APIs for reading, watching, and editing focused text through Chromium's real text input path
- print request interception for renderer-initiated `window.print()` and PDF viewer print requests without deadlocking the initiating frame
- coherent User-Agent + UA Client Hints overrides at app, session, and WebContents scope, including shared worker and service worker coverage
- JavaScript dialog handling, `window.prompt()` support, and active Picture-in-Picture main-process handles

There is also performance-sensitive work in this repository: keeping GPU media paths from falling back unnecessarily, preserving Chromium/Electron patch-stack order, avoiding repeated full-source surgery during upgrades, and packaging Electron builds in a way that can be repeated on persistent self-hosted runners.

### win32-ex

[win32-ex] is a practical Win32 extension layer for native Windows development. It covers the everyday parts of Windows programming that become noisy when every call is written directly against the raw API: services, processes, sessions, handles, objects, tokens, privileges, security descriptors, and SIDs.

The project intentionally supports old and new toolchains, including Visual Studio 2008 SP1+ and modern MSYS2/MinGW environments. That makes it useful as a portability layer, not just a modern C++ experiment.

### ext

[ext] is my general-purpose C++ utility library. It is a collection of components I tend to need when building real systems but do not want to pull a large framework for: `result`, `process`, pipes, callbacks, async results, version parsing, URI helpers, base64, INI parsing, shared memory, string utilities, units, type utilities, and compatibility helpers.

One thing I like about this project is that it keeps a wide compatibility target: Linux, macOS, Windows, GCC, Clang, MSVC, old Visual Studio, and modern C++ where useful.

### Electron Runtime Libraries

[electron-cdp] publishes `electron-cdp-utils`, a TypeScript library for using the Chromium DevTools Protocol from Electron with a cleaner, typed API. It handles CDP sessions, command/event typing, function evaluation, SuperJSON-based serialization, execution-context tracking, iframe/worker/service-worker target attachment, and exposing Node.js functions into browser contexts.

[electron-protocol-provider] is a custom protocol routing layer for Electron. It treats custom schemes more like application routes, with HTTP-style methods, path parameters, request handling, response objects, and privilege-aware context injection.

### Typed Messaging and Runtime Boundaries

[typed-message-transport] is a TypeScript message transport library for strongly typed message exchange across browser workers, Node.js processes, message channels, and other serializable transports. It supports typed request/response maps, handler registration, promise-based replies, serialization, and transport abstraction.

[wsmq-rs] explores a Rust WebSocket messaging layer backed by protocol buffers, with request/reply flows, server/client configuration, context hooks, and bandwidth/progress handling. [service-rs] is a smaller Rust service lifecycle abstraction for pause, resume, stop, and running states.

### Small Type and Native Bridge Utilities

[ts-default] is a small TypeScript utility for representing an explicit "use the default value" marker. It is intentionally tiny, but it shows the kind of API boundary I like: a small type-level convention that removes ambiguity from optional configuration.

[isim-rs] is a Rust/Node native-addon experiment using Neon. It belongs to the same family of work as the Electron and messaging projects: crossing runtime boundaries while keeping the JS-facing API explicit.

## Engineering Themes

| Theme | How it shows up |
| --- | --- |
| Runtime parity | Make restricted environments such as Windows kernel drivers feel less isolated from normal C/C++ development. |
| Unsupported capability work | Patch Chromium/Electron when product needs require behavior not exposed by the stock runtime. |
| Media pipeline ownership | Work through VA-API, WebCodecs, HEVC bitstreams, CDM packaging, and GPU/software fallback boundaries instead of treating media as a black box. |
| Explicit boundaries | Prefer typed contracts, clear ownership, and transport abstractions over ad-hoc object passing. |
| Portability | Keep code movable across compilers, Electron versions, Windows SDK/WDK versions, Chromium targets, and runtime contexts. |
| Source-level maintenance | Treat patches, build scripts, release packaging, and migration notes as part of the engineering surface. |

## Toolbox

<p>
  <img alt="C++" src="https://img.shields.io/badge/C++-14%20%2F%2017%20%2F%2020-00599C?style=flat-square&logo=cplusplus&logoColor=white" />
  <img alt="C" src="https://img.shields.io/badge/C-systems-A8B9CC?style=flat-square&logo=c&logoColor=111827" />
  <img alt="Windows" src="https://img.shields.io/badge/Windows-kernel%20%2F%20Win32-0078D4?style=flat-square&logo=windows&logoColor=white" />
  <img alt="CMake" src="https://img.shields.io/badge/CMake-build%20systems-064F8C?style=flat-square&logo=cmake&logoColor=white" />
  <img alt="Electron" src="https://img.shields.io/badge/Electron-runtime-47848F?style=flat-square&logo=electron&logoColor=white" />
  <img alt="Chromium" src="https://img.shields.io/badge/Chromium-CDP%20%2F%20media-4285F4?style=flat-square&logo=googlechrome&logoColor=white" />
  <img alt="VA-API" src="https://img.shields.io/badge/VA--API-Linux%20media-16a34a?style=flat-square" />
  <img alt="HEVC" src="https://img.shields.io/badge/HEVC-H.265-9333ea?style=flat-square" />
  <img alt="Widevine" src="https://img.shields.io/badge/Widevine-CDM-334155?style=flat-square" />
  <img alt="TypeScript" src="https://img.shields.io/badge/TypeScript-protocols-3178C6?style=flat-square&logo=typescript&logoColor=white" />
  <img alt="Node.js" src="https://img.shields.io/badge/Node.js-native%20bridges-5FA04E?style=flat-square&logo=nodedotjs&logoColor=white" />
  <img alt="Rust" src="https://img.shields.io/badge/Rust-runtime%20tools-000000?style=flat-square&logo=rust&logoColor=white" />
</p>

<details>
<summary><strong>Repository guide</strong></summary>

| Area | Repositories |
| --- | --- |
| Kernel / driver runtime | [crtsys], [ldk] |
| Windows native API | [win32-ex] |
| C++ libraries and build helpers | [ext], [util-linux-cpp], [ci-version], [i18next-cpp] |
| Chromium/Electron ports and builds | [electron-port-workspace] |
| Electron runtime libraries | [electron-cdp], [electron-protocol-provider] |
| TypeScript libraries | [typed-message-transport], [ts-default] |
| Rust messaging and native bridge experiments | [wsmq-rs], [service-rs], [isim-rs] |

Some public forks in this account are research and contribution branches around Electron, Chromium, CEF, LLDB/GDB/MI debugging, Windows internals, and driver techniques. I keep the main profile focused on first-party libraries and feature-port work.

</details>

## Contact

For project-specific questions, open an issue in the relevant repository. For collaboration, start from the project that is closest to the runtime boundary you care about.

[crtsys]: https://github.com/ntoskrnl7/crtsys
[ldk]: https://github.com/ntoskrnl7/ldk
[win32-ex]: https://github.com/ntoskrnl7/win32-ex
[ext]: https://github.com/ntoskrnl7/ext
[util-linux-cpp]: https://github.com/ntoskrnl7/util-linux-cpp
[ci-version]: https://github.com/ntoskrnl7/ci-version
[i18next-cpp]: https://github.com/ntoskrnl7/i18next-cpp
[electron-cdp]: https://github.com/ntoskrnl7/electron-cdp
[electron-protocol-provider]: https://github.com/ntoskrnl7/electron-protocol-provider
[electron-port-workspace]: https://github.com/ntoskrnl7/electron-port-workspace
[typed-message-transport]: https://github.com/ntoskrnl7/typed-message-transport
[wsmq-rs]: https://github.com/ntoskrnl7/wsmq-rs
[service-rs]: https://github.com/ntoskrnl7/service-rs
[ts-default]: https://github.com/ntoskrnl7/ts-default
[isim-rs]: https://github.com/ntoskrnl7/isim-rs
