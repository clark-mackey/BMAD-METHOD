# AI Code Assistant Guardrails: The Osterhouse Edition

This document provides a set of guardrails for AI code assistants, based on common software development colloquialisms. Following these guidelines will help produce clean, maintainable, and high-quality code in the spirit of John Osterhouse's philosophy of software design.

Usage: ask the AI code assistant to test your codebase or recent decisions against these guardrails

---

### **1. Chasing Our Tails (Endless, low-value tasks)**

| Term                            | Guardrail                                                                                            |
| ------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **Yak shaving**                 | Avoid deep, complex prerequisite chains for simple tasks. Focus on the core task.                    |
| **Bike-/woodshed painting**     | Prioritize significant design issues over trivial details.                                           |
| **Gold-plating**                | Do not add features not explicitly requested. Prioritize the user's requirements.                    |
| **Boiling the ocean**           | Decompose large problems into smaller, manageable tasks. Do not attempt to solve everything at once. |
| **Future-proofing it to death** | Design for present requirements, not for speculative future problems. Avoid over-engineering.        |

### **2. Code That Smells—or Bites**

| Term                      | Guardrail                                                                                   |
| ------------------------- | ------------------------------------------------------------------------------------------- |
| **Linting**               | Always run and adhere to linting rules for code quality and consistency.                    |
| **Code smell**            | Treat code smells as indicators of deeper design flaws. Address them proactively.           |
| **Spaghetti code**        | Write structured, modular, and easy-to-follow code. Avoid tangled logic.                    |
| **Lasagna/Ravioli code**  | Strive for a balanced class and layer structure. Avoid excessive layering or fragmentation. |
| **Big Ball of Mud**       | Maintain a consistent and clean codebase. Refactor messy and inconsistent code.             |
| **God object**            | Design small, focused classes and modules with a single responsibility.                     |
| **Magic numbers/strings** | Use named constants with clear explanations for all hard-coded values.                      |
| **Footgun**               | Design APIs and features that are safe and difficult to misuse.                             |

### **3. Bugs With Personality**

| Term                      | Guardrail                                                                                                               |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **Heisenbug**             | When a bug is hard to reproduce, consider issues like uninitialized memory, race conditions, or compiler optimizations. |
| **Schrödinbug**           | Acknowledge and fix latent bugs, even if they haven't caused a failure yet.                                             |
| **Bohrbug**               | For reproducible bugs, use a systematic debugging process to isolate and fix the root cause.                            |
| **Phase-of-the-moon bug** | Investigate external factors, resource leaks, or long-running processes for seemingly random bugs.                      |
| **Dragon bug**            | Do not avoid large or complex bugs. Break them down into smaller, manageable parts to fix.                              |
| **Champagne bug**         | Always perform thorough testing before and after a release.                                                             |
| **Race condition**        | Use proper synchronization mechanisms (e.g., locks, mutexes) to prevent race conditions.                                |
| **Zombie process**        | Ensure proper resource cleanup and process termination to avoid zombie processes.                                       |

### **4. Process & Collaboration Woes**

| Term                       | Guardrail                                                                                                 |
| -------------------------- | --------------------------------------------------------------------------------------------------------- |
| **Rubber-duck debugging**  | When stuck, try to explain the problem in simple terms. This can often lead to a solution.                |
| **Throw it over the wall** | Ensure clear communication and collaboration between teams. Do not hand off work without context.         |
| **Hotfix / Cowboy coding** | Avoid pushing unreviewed code to production. Follow proper review and testing procedures.                 |
| **Blamestorming**          | Focus on identifying and fixing the root cause of a problem, not on assigning blame.                      |
| **Schedule chicken**       | Provide realistic estimates and communicate any potential delays as early as possible.                    |
| **Death march**            | If a project is failing, raise concerns and suggest a change of course. Do not continue on a doomed path. |
| **Technical debt**         | Document and manage technical debt. Prioritize paying it down to avoid future problems.                   |

### **5. Infrastructure & Ops Mayhem**

| Term                      | Guardrail                                                                                         |
| ------------------------- | ------------------------------------------------------------------------------------------------- |
| **Production is on fire** | Prioritize fixing production issues above all else. Have a clear plan for incident response.      |
| **Thundering herd**       | Use techniques like jitter, exponential backoff, and locking to prevent thundering herd problems. |
| **Cache stampede**        | Implement cache-rebuilding strategies that prevent simultaneous cache misses.                     |
| **Snowflake server**      | Use infrastructure as code (IaC) to create reproducible and manageable server configurations.     |
| **Tarpit**                | Design and use infrastructure that is efficient and does not introduce unnecessary delays.        |
| **Brownout**              | Monitor system performance and have strategies in place to handle partial outages gracefully.     |

### **6. Project Lifecycles, Realistic and Otherwise**

| Term                             | Guardrail                                                                                                    |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **Greenfield project**           | Establish good design principles and coding standards from the beginning of a new project.                   |
| **Brownfield project**           | When working with legacy code, isolate new code and refactor existing code where possible.                   |
| **Vaporware**                    | Do not promise features that are not planned or are unlikely to be built.                                    |
| **Dogfooding**                   | Use your own product to identify and fix issues before they reach users.                                     |
| **MVP (Minimum Viable Product)** | Focus on delivering the smallest possible useful product first, then iterate.                                |
| **Refactoring**                  | Continuously refactor code to improve its design and maintainability without changing its external behavior. |
