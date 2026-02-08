https://github.com/chirag-13joy/sandboxed-python/releases

[![Releases](https://img.shields.io/badge/Releases-Visit%20Now-blue?logo=github&logoColor=white)](https://github.com/chirag-13joy/sandboxed-python/releases)

# Sandboxed Python: A Safe, Finite Python Sandbox for Secure Execution

![Python Logo](https://www.python.org/static/community_logos/python-logo.png)

A lightweight runtime that runs a restricted subset of Python. It lets you execute code with clear limits on resources and scope. The goal is to provide a clean, predictable environment for untrusted code, while still supporting a useful subset of Python features.

This project focuses on safety, determinism, and simplicity. It offers a practical sandbox that is easy to reason about, easy to test, and easy to extend. It is not a full security fortress. It is a safe tool for learning, experimentation, and controlled execution in environments where untrusted code should not affect the host.

Table of contents
- Overview
- Goals and design principles
- What’s supported
- How the sandbox works
- Getting started
- Installation
- Quick start examples
- API reference
- Architecture and internals
- Safety and limitations
- Running examples and tests
- Development and contribution
- Releases and versioning
- FAQ
- License

Overview
The sandboxed Python project provides a real, runnable Python sandbox that is finite and safe by design. It isolates execution, limits resources, and restricts what untrusted code can do. It focuses on providing a practical subset of Python, not the entire language. The runtime is compact, lean, and easy to inspect. It aims to balance usefulness with strong, defensible boundaries.

This README uses plain language and concrete examples. It aims to help new users understand what the project does, how to try it, and how to contribute. It also helps teams think about when to reach for a sandbox like this and how to integrate it into their tools and pipelines.

Goals and design principles
- Safety first: The sandbox reduces risk by removing or neutralizing dangerous features. It blocks direct file system access, dangerous imports, and unrestricted runtime introspection.
- Finite execution: The sandbox imposes limits on CPU time and memory. It also constrains loops and growth to prevent runaway code.
- Predictability: The environment is deterministic under a given input. It avoids hidden side effects that would surprise users.
- Usability: The subset is chosen to be practical. It supports common tasks while staying easy to reason about.
- Extensibility: The codebase is modular. It is straightforward to plug in new restrictions, new builtins, or new evaluation strategies.
- Transparency: The architecture is observable. Users can inspect sandbox state, limits, and the executed code path.

What’s supported
- Core data types: numbers, strings, booleans, lists, dictionaries, sets, tuples.
- Basic control flow: if/else, for loops, while loops with iteration guards, break and continue.
- Functions: user-defined functions, closures with bounded scope, and simple lambda support.
- Arithmetic and comparisons: standard operators with predictable semantics.
- Builtins: a restricted set of safe builtins (e.g., len, range, sorted) is available. Dangerous builtins (os, sys, open, eval, exec) are blocked or replaced with safe stubs.
- Modules: standard library access is strictly limited. Only a curated, safe subset of modules is permitted, and only through a controlled import mechanism.
- I/O: no direct file system or network access. Output is captured and returned to the caller.
- Environment: a clean global namespace with a restricted __builtins__ and no access to the host environment.

What’s not supported
- Full Python runtime: the sandbox does not execute every edge case of the language. Some dynamic features are disabled to preserve safety.
- Arbitrary foreign libraries: external C extensions, dynamic imports of arbitrary modules, and low-level system access are not allowed.
- Real-time system calls: any code that attempts to interact with the host system beyond safe, captured outputs is blocked.
- Multi-threading hazards: concurrent execution is either disabled or strictly controlled to avoid races and resource contention.

How the sandbox works
- Restricted execution environment: The sandbox runs code inside a controlled interpreter loop with a carefully curated global and local namespace.
- Safe builtins and banned operations: A whitelist of safe builtins is provided. All other builtins are blocked or proxied with safety checks.
- Resource guards: CPU time and memory usage are bounded. The sandbox enforces limits to prevent runaway processes.
- Import controls: Import is mediated. Only approved modules can be loaded, and imports run in a restricted context.
- Code analysis and sizing: Before execution, code can be inspected for disallowed constructs. A size or complexity cap helps keep the workload in check.
- Output capture: All standard output and error streams are captured. The host receives results, including printed data, as structured results.
- Isolation boundaries: The sandbox follows a layered approach. Each layer reduces the risk of side effects leaking into the host.

Getting started
- This sandbox is built for developers who want to run untrusted code safely, or for teams that need bounded Python execution as part of a larger system.
- It is well suited for teaching environments, online judges, code runners, and sandboxed experimentation.
- It integrates into CI pipelines, testing tools, and security tutorials where safe code execution matters.

Installation
- Prerequisites: Python 3.8+ and a modern operating system with standard toolchains installed.
- Clone the repository:
  - git clone https://github.com/chirag-13joy/sandboxed-python.git
  - cd sandboxed-python
- Install dependencies:
  - python -m pip install -r requirements.txt
- Build or set up the package for your environment following the instructions in the repository’s setup documentation.
- It is common to install in editable mode during development:
  - python -m pip install -e .

Note: The exact commands may vary based on your platform. The repository’s installation guide provides the precise steps for Linux, macOS, and Windows.

Quick start examples
- The quick start examples show a minimal workflow to run untrusted code inside the sandbox.

Example 1: Run a simple script
- Python code:
  - print("Hello from the sandbox!")
- How to run:
  - from sandboxed_python.sandbox import Sandbox
  - sandbox = Sandbox(max_time_s=2.0, max_memory_mb=256)
  - code = 'print("Hello from the sandbox!")'
  - result = sandbox.run(code)
  - print("Output:", result.output)

Example 2: Perform a safe computation
- Python code:
  - def compute(n):
      total = 0
      for i in range(n):
          total += i
      return total
  - print(compute(10))
- How to run:
  - result = sandbox.run(code)
  - print("Result:", result.output or result.value)

Example 3: Handle errors gracefully
- Python code:
  - try:
      x = 1 / 0
    except Exception as e:
      print("Error:", type(e).__name__)
- How to run:
  - result = sandbox.run(code)
  - print("Error captured:", result.error)

Example 4: Restricting imports
- Python code:
  - import os
  - print(os.listdir("."))
- Expected: Import is blocked or redirected to a safe stub.
- How to run:
  - result = sandbox.run(code)
  - print("Status:", result.status)

These examples demonstrate the basics of interacting with the sandbox. They illustrate how to feed code, capture output, and handle errors. You can adjust the sandbox parameters to tune time and memory constraints for your use case.

API reference
- Sandbox: The main entry point for executing code in a safe, controlled environment.
  - Constructor:
    - Sandbox(max_time_s: float, max_memory_mb: int, allowed_imports: Optional[List[str]] = None)
  - Methods:
    - run(code: str) -> ExecutionResult
      - Executes the given code string in the sandbox.
      - Returns an ExecutionResult with fields:
        - output: str or None (captured standard output)
        - error: str or None (captured error message)
        - return_value: any (the value produced by the code, if any)
        - status: str (e.g., "ok", "error", "timeout", "memory_error")
        - time_ms: int (execution time in milliseconds)
        - memory_mb: int (peak memory usage in MB)
- ExecutionResult: A simple container for results.
  - Attributes:
    - output
    - error
    - return_value
    - status
    - time_ms
    - memory_mb

Architecture and internals
- Layered design: The sandbox uses a layered approach to separate concerns.
  - Parsing and validation layer checks the code for disallowed constructs and measures code size.
  - Secure execution layer runs the code inside a restricted environment with a carefully curated namespace and builtins.
  - Resource control layer enforces time and memory quotas and ensures code cannot exceed its budget.
  - I/O layer captures all standard output and error and provides it to the host in a structured form.
- Code restriction strategy:
  - Only a subset of Python features is allowed.
  - Imports are controlled. Only safe modules can be loaded.
  - Builtins are replaced with safe wrappers or disabled entirely.
  - Dangerous features like eval, exec, and direct system calls are blocked.
- Determinism and reproducibility:
  - The sandbox uses fixed seeds for any non-deterministic components, ensuring consistent behavior for the same input.
- Observability:
  - Users can query the sandbox for resource usage and internal state to understand how code executes.

Safety and limitations
- The sandbox provides strong boundaries for many common risk scenarios but is not a foolproof security tool.
- It helps prevent accidental or careless code from harming the host.
- It is ideal for teaching, testing, prototyping, and controlled experiments.
- It should not be used as the sole defense in high-security contexts. For critical security needs, complement with additional layers of protection and reviews.

Running examples and tests
- The repository includes a suite of tests and example files.
- To run tests:
  - pytest -q
- To run a simple example script:
  - python -m sandboxed_python.examples.run_example --script path/to/script.py
- If you run into issues, consult the test coverage reports and the debug logs produced by the sandbox. They contain helpful details about what the sandbox allows and blocks.

Development and contribution
- How to contribute:
  - Start with the issues tab to find low-friction tasks.
  - Fork the repository, implement changes in a feature branch, and open a pull request.
  - Write tests for new features or bug fixes.
  - Keep code style consistent with the project guidelines.
- Coding style:
  - Use clear, direct sentences.
  - Prefer simple constructs over complex ones.
  - Use descriptive names for functions and variables.
  - Document non-obvious behavior with short comments.
- Testing approach:
  - Add unit tests for new features.
  - Include regression tests for bugs you fix.
  - Ensure tests cover edge cases and typical usage.
- Documentation:
  - Update the README with any new APIs.
  - Provide examples that illustrate real-world usage.
  - Keep installation instructions accurate for major platforms.

Releases and versioning
- The project follows semantic versioning where possible.
- The Releases page hosts build artifacts, release notes, and installable bundles.
- See the Releases page to learn what changed in each version.
- For quick access, the releases page is linked here: https://github.com/chirag-13joy/sandboxed-python/releases

FAQ
- What is the goal of this sandbox?
  - To provide a safe, finite environment to run a practical subset of Python without risking the host system.
- What happens if the code tries to do something unsafe?
  - The sandbox blocks the action. If it detects an attempt to bypass the limits, it reports the event and stops execution.
- Can I customize what is allowed?
  - Yes. The sandbox can be configured with a custom set of allowed imports and builtins. You can extend it to fit your needs while maintaining safety boundaries.
- Is the sandbox fully secure?
  - It provides strong protection for common risk scenarios. It is not a guarantee against all security threats. Use it as part of a defense-in-depth strategy.

Usage scenarios
- Education and tutorials:
  - Demonstrate Python concepts in a controlled environment.
  - Allow learners to run code without touching the host system.
- Online coding platforms:
  - Run user submissions in a bounded sandbox to prevent abuse.
- Research and experimentation:
  - Explore how Python code behaves under constraints.
  - Test ideas without risking the host environment.
- Bind the sandbox to automated tasks:
  - Use it to execute small code snippets as part of larger pipelines.

Key design decisions
- Finite resources:
  - Time and memory limits are set to prevent long or memory-heavy runs.
- Restricted surface:
  - The global namespace is sanitized, and only safe builtins are provided.
- Predictable behavior:
  - The sandbox minimizes surprises by enforcing clear rules and returning structured results.
- Extensibility:
  - The architecture is modular, so adding new restrictions or features is straightforward.

Operational considerations
- Performance:
  - The sandbox trades some raw speed for safety. For typical educational tasks, this is a fair trade.
- Debugging:
  - The sandbox provides helpful error messages and stack traces. It logs disallowed operations to aid debugging.
- Portability:
  - The code is designed to work across common operating systems with Python support. Some platform-specific behavior may require tuning.

Code examples and patterns
- Safe function execution pattern:
  - Define a function inside the sandbox, then call it with controlled arguments.
  - The sandbox ensures no external access or side effects leak out.
- Capturing results:
  - Every run returns a structured ExecutionResult object. It contains output, errors, and metadata about the run.

Sample usage snippets (for documentation)
- Basic run:
  - from sandboxed_python.sandbox import Sandbox
  - sandbox = Sandbox(max_time_s=2.0, max_memory_mb=256)
  - code = 'print("safe run")'
  - result = sandbox.run(code)
  - print(result.output)
- Handling errors:
  - code = '1 / 0'
  - result = sandbox.run(code)
  - assert "ZeroDivisionError" in result.error

Advanced usage and customization
- Custom allowed imports:
  - You can pass a list of module names that are safe to import.
  - The sandbox will only permit those modules and will block others.
- Adjusting resource limits:
  - Increase memory for larger tasks if your environment supports it.
  - Tune the time limit to balance responsiveness and task complexity.
- Instrumentation:
  - The sandbox exposes hooks to observe and log key events during execution.
  - You can attach your own observers to monitor code paths and resource usage.

Performance notes
- The sandbox is designed for bounded workloads. It avoids heavy dynamic analysis during run time to keep overhead predictable.
- If you run many short scripts, the initialization cost can dominate. Consider reusing a sandbox instance for batch tasks to minimize setup overhead.
- For large workloads, measure memory usage and time carefully. Adjust limits to fit your environment.

Community and ecosystem
- The project welcomes contributions from learners, educators, researchers, and developers who care about safe code execution.
- If you have ideas for new features, or run into issues, you can open an issue on GitHub and start a discussion.
- You may link to relevant educational resources, examples, or compatibility notes in related projects.

Image and visual assets
- The README includes a Python logo to reinforce the theme of Python execution in a safe sandbox.
- You can include additional visuals in future releases, such as:
  - A schematic of the sandbox architecture.
  - A diagram showing the flow from user code to sandboxed execution to result.
  - Illustrations demonstrating safe boundaries and resource limits.

Why you might choose this sandbox
- You need a controlled playground for untrusted code.
- You want predictable behavior for teaching and testing.
- You require a clear boundary between code execution and host resources.
- You prefer a lean, maintainable implementation with straightforward extension points.

Roadmap
- Expand the supported subset with more common Python patterns while preserving safety constraints.
- Improve the import sandbox by widening the safe module set with strict checks.
- Add richer debugging support, including step-wise execution and introspection tools.
- Introduce more robust diagnostics and observability to help users understand sandbox behavior.
- Improve cross-platform compatibility, especially for Windows and non-Linux environments.
- Create integration adapters for common hosting environments like Jupyter, CI systems, and online judges.

License
- The project is released under an open license that encourages adoption, adaptation, and contribution.
- The license terms support both personal and commercial use with standard attribution requirements.
- Always refer to the LICENSE file in the repository for the exact terms.

Releases and releases notes
- The Releases section contains downloadable bundles and release notes for each version.
- It provides a clear history of changes, bug fixes, and feature additions.
- To review the latest changes, visit the Releases page: https://github.com/chirag-13joy/sandboxed-python/releases
- For direct access to assets, you can browse the Releases page to locate installers, packages, or code samples that fit your setup.
- If you need a specific artifact, this is the primary place to find it and to verify compatibility with your environment.

Compatibility and platform notes
- Linux: Works well with standard kernels and common toolchains.
- macOS: Should function with common Python distributions; ensure you have the right permissions for resource control features.
- Windows: Supported in modern environments; some Linux-specific assumptions may require adjustments.

Common gotchas
- Resource limits: If you set very tight limits, simple code may hit the boundary. Adjust the time and memory settings accordingly.
- Import restrictions: A script that relies heavily on imports of many modules may fail due to the safe module policy. Plan for incremental enhancements to allowed modules if needed.
- Networking: If the untrusted code attempts to access the network, the sandbox blocks it. Ensure automation expects this behavior.

Maintenance and governance
- The project follows a lightweight governance model. Changes go through an issue and PR workflow.
- Maintainers review changes for safety, correctness, and clarity.
- New contributors are welcome to start with small, well-scoped issues.

Documentation and learning resources
- The README serves as a starting point for understanding the sandbox.
- Additional docs cover installation specifics, API details, and advanced usage.
- Tutorials illustrate common patterns for safe code execution and testing in isolation.
- Example scripts demonstrate real-world usage, such as sandboxed code evaluation and result collection.

Security posture and best practices
- The sandbox aims to minimize risk by design. It blocks dangerous operations and isolates execution.
- For high-risk environments, complement with additional layers of defense and monitoring.
- Always review the generated results and logs to ensure the sandbox behaved as expected.

Summary of usage patterns
- Quick testing of small code snippets in a controlled environment.
- Safe evaluation of user-submitted code prior to broader execution.
- Educational demos that illustrate Python language features without exposing the host to danger.
- Research experiments exploring the behavior of Python code under restricted conditions.

Appendix: deeper dive into the sandbox loop
- The core loop reads input code, validates it, compiles it with safety checks, and then executes within a restricted namespace.
- The result is collected and returned to the caller with a structured summary of outputs, errors, and resource usage.
- The design favors a predictable path: validate, sandbox, run, report, reset.

Appendix: customization ideas
- Extend the allowed builtins with a carefully curated set that your project requires.
- Add domain-specific restrictions for environments like data science pipelines or teaching labs.
- Create tailored templates for common tasks to reduce boilerplate in user code.

Enduring principles
- Safety, simplicity, and clarity guide every design decision.
- The sandbox remains approachable for newcomers while offering enough depth for advanced users.
- The project remains focused on practical use cases over theoretical depth.

Link recap
- The primary release page for this project is https://github.com/chirag-13joy/sandboxed-python/releases. This page hosts release notes, assets, and installable bundles.
- For quick access to the latest release and its details, you can visit the link above. The same URL can be found at the top of this document and again later in the Releases section to help you locate assets and notes quickly.