# Top 10 Actionable Rules from Three Software Design Masters

Here are 10 clear, actionable rules from the actual writings of John Ousterhout, Robert C. Martin, and Martin Fowler, based on their documented principles and guidelines from their books and published work.

---

## John Ousterhout

*From "A Philosophy of Software Design"*

1.  **Complexity is the Enemy**: The most fundamental problem in software design is managing complexity. Anything that makes a system hard to understand or modify is complexity. Your primary goal is to minimize it.

2.  **Modules Should Be Deep**: A deep module is one that has a simple interface but a powerful implementation. It hides a lot of complexity behind that simple interface.

3.  **Avoid Shallow Modules**: A shallow module is one with a complex interface but a simple implementation. These modules don't hide complexity; they just expose it.

4.  **Pull Complexity Downwards**: It is more important for a module to have a simple interface than a simple implementation. It's better for a module to suffer a little internal complexity than to inflict that complexity on its users.

5.  **Design It Twice**: Don't just go with your first idea. Try to come up with a couple of different designs for a new feature or system. The second one is often better.

6.  **Information Hiding is Key**: Hide implementation details within a module. This reduces dependencies between modules and makes the system easier to change.

7.  **Define Errors Out of Existence**: The best way to handle errors is to design your system in such a way that they can't happen in the first place.

8.  **Write Comments First**: Before you write the code for a new method or class, write a comment that describes what it does. This will help you to clarify your thinking and to create a better design.

9.  **Don't Optimize Prematurely**: Don't try to optimize your code until you know that it's a performance bottleneck. Measure first, then optimize.

10. **Good Code is Not Enough**: Good code is not enough. You also need good documentation. Comments should describe the things that are not obvious from the code itself.

---

## Robert C. Martin (Uncle Bob)

*From "Clean Code" and the SOLID Principles*

1.  **Single Responsibility Principle (SRP)**: A class should have only one reason to change. Gather together the things that change for the same reasons. Separate things that change for different reasons.

2.  **Open/Closed Principle (OCP)**: Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.

3.  **Liskov Substitution Principle (LSP)**: Subtypes must be substitutable for their base types. A program that uses an interface must not be confused by an implementation of that interface.

4.  **Interface Segregation Principle (ISP)**: Clients should not be forced to depend on interfaces they do not use. Keep interfaces small and focused.

5.  **Dependency Inversion Principle (DIP)**: High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions.

6.  **The Boy Scout Rule**: Leave the code cleaner than you found it. Every time you touch a piece of code, you should make a small improvement to it.

7.  **Functions Should Be Small and Do One Thing**: Functions should be small, and they should do one thing. They should do it well. They should do it only.

8.  **Use Descriptive Names**: The name of a variable, function, or class should answer all the big questions. It should tell you why it exists, what it does, and how it is used.

9.  **Don't Repeat Yourself (DRY)**: Every piece of knowledge must have a single, unambiguous, authoritative representation within a system.

10. **Tests are First-Class Citizens**: Tests are not an afterthought. They are a fundamental part of the development process. Write tests before you write the code (TDD).

---

## Martin Fowler

*From "Refactoring" and his writings on software architecture*

1.  **The Two Hats Principle**: When you're programming, you should be wearing one of two hats: the refactoring hat or the feature-adding hat. When you're refactoring, you're not adding new functionality. When you're adding new functionality, you're not refactoring.

2.  **The Rule of Three**: When you're doing something for the first time, just do it. When you're doing it for the second time, you'll cringe and do it again. When you're doing it for the third time, you refactor.

3.  **Refactor When You Fix a Bug**: Bugs are a good time to refactor. When you find a bug, it's often a sign that the code is not clear enough. Use the opportunity to clean it up.

4.  **Architecture is About the Important Stuff**: Focus your architectural efforts on the things that are hard to change. Don't get bogged down in the details that are easy to change later.

5.  **High Internal Quality Leads to Faster Delivery**: A high-quality codebase is easier to change. This means that you can add new features more quickly. Don't sacrifice quality for speed.

6.  **Beck's Four Rules of Simple Design**: 1. Passes the tests. 2. Reveals intention. 3. No duplication. 4. Fewest elements. These rules, in priority order, are a great guide to simple design.

7.  **Separate Presentation from Domain Logic**: Keep the code that deals with the user interface separate from the code that deals with the business logic. This makes the system easier to test and to change.

8.  **Refactoring Should Be a Continuous Activity**: Refactoring is not something you do once in a while. It's something you do all the time, in small steps.

9.  **Don't Refactor Close to a Deadline**: If you're close to a deadline, it's not the time to start a major refactoring. Wait until after the deadline, and then make the changes you need to make.

10. **When Code is Too Messy, Rewrite It**: Sometimes, the best thing to do with a messy piece of code is to throw it away and start over. Don't be afraid to rewrite code when it's necessary.

