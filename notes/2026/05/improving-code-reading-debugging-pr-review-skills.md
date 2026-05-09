---
title: "Improving Code Reading, Debugging, & PR Review Skills"
slug: "improving-code-reading-debugging-pr-review-skills"
tags: ["debugging", "code-reading", "pr-review", "learning", "fundamentals"]
summary: "This note provides resources and practices to strengthen fundamental code-reading, debugging, and PR review skills, crucial for becoming an irreplaceable developer."
created: 2026-05-09
updated: 2026-05-09
source_question: "One thing, I am bad at debugging reading code and PR reviews, provide me some good YouTube sources, reading practices, books with which I can improve"
review:
  last_reviewed: null
  next_review: 2026-05-09
  step: 0
  confidence: 0
quiz:
---

Improving your code-reading, debugging, and PR review skills are foundational to becoming a strong, indispensable developer. These aren't about speed, but about depth of understanding.

Here are some resources and practices tailored to your goals:

### **Reading Practices (The Core Skill)**

This is where the rubber meets the road. Consistent practice is key.

1.  **Start Small & Focused:** Don't try to understand an entire codebase at once. Pick a single function, a small module, or a specific feature.
2.  **Trace Execution Mentally (or on Paper):**
    *   **Line-by-Line:** Pretend you are the computer. What is the value of each variable *after* this line executes? What's the next line?
    *   **Input-Output Mapping:** For a function, what inputs will produce what outputs? How does the code transform the input?
    *   **Draw Diagrams:** For complex logic or data structures, sketch out the flow (flowchart) or the data shape (ER diagram for objects/data, class diagram for types).
3.  **Ask "Why?":**
    *   Why is this variable named this way?
    *   Why is this `if` statement here? What problem does it solve?
    *   Why this specific data structure (array vs. map vs. set)? What are the trade-offs?
    *   Why are these lines in this order? What would break if they were reordered?
    *   Why isn't this handled by a library function?
4.  **Refactor in Your Head (or a Scratchpad):** Once you understand a piece of code, think: "How would I write this differently? What are the pros/cons of my alternative?" This builds critical thinking about code quality.
5.  **Read the Tests:** Tests often reveal the intended behavior and edge cases of the code. They are a form of documentation.
6.  **Read the Git History:** `git blame` or `git log -p <file>` can show *when* a line was added/changed and *why* (via commit messages). This adds crucial context.
7.  **Read Documentation (and Write It):** If documentation exists, read it. If it doesn't, try to write some for the code you're reading – this forces a deeper understanding.

### **Debugging Practices**

Debugging is applied code-reading under pressure.

1.  **Reproduce Consistently:** Before you touch any code, ensure you can reliably reproduce the bug.
2.  **Isolate the Problem:**
    *   **Binary Search Debugging:** If you have a large block of code, comment out half of it. Does the bug still occur? Repeat until you pinpoint the problematic section.
    *   **Smallest Possible Example:** Can you create a minimal code snippet that *only* exhibits the bug? This helps eliminate noise.
3.  **Use Your Tools:**
    *   **Debugger (IDE/Browser DevTools):** Learn breakpoints, stepping through code (step over, step into, step out), watching variables, and the call stack. This is non-negotiable for serious debugging.
    *   **Logging:** Judicious `console.log` (or equivalent) can be powerful for tracing values and execution flow.
4.  **Formulate Hypotheses:** Don't just randomly change things. Based on your code-reading, form a hypothesis about *why* the bug is happening, then use your debugger/logs to test it.
5.  **Explain the Bug (Rubber Duck Debugging):** Explain the problem, the expected behavior, and the actual behavior to an inanimate object (or a person). The act of explaining often reveals the flaw in your own logic.

### **PR Review Practices**

PR reviews are about applying your code-reading and critical thinking skills to someone else's work, ensuring quality and maintainability.

1.  **Understand the Goal:** Read the PR description and associated ticket. What problem is this PR trying to solve?
2.  **High-Level First Pass:** Skim the entire change. Does the overall approach make sense? Are there any obvious architectural concerns?
3.  **Detailed Line-by-Line Review:**
    *   **Correctness:** Does the code do what it's supposed to? Are there edge cases missed?
    *   **Readability:** Is it easy to understand? Are variable/function names clear?
    *   **Maintainability:** Will this be easy to extend or debug in the future?
    *   **Performance:** Are there any obvious performance pitfalls?
    *   **Security:** Any potential vulnerabilities introduced?
    *   **Tests:** Are there adequate tests for the new/changed functionality? Do they cover edge cases?
    *   **Adherence to Standards:** Does it follow the team's coding style and conventions?
4.  **Ask Questions, Don't Just State Problems:** Instead of "This is wrong," try "Could you explain why `X` is done this way instead of `Y`? I'm concerned about `Z`." This fosters learning and collaboration.
5.  **Suggest Improvements, Don't Rewrite:** Offer concrete, actionable suggestions rather than just pointing out flaws.
6.  **Review the "Diff," Not Just the Files:** Focus on what *changed*. Sometimes the context of removed lines is important.

### **YouTube Sources**

Focus on channels that explain *how* things work, not just *what* they do.

*   **For JavaScript/Web Fundamentals:**
    *   **"Fun Fun Function" (Mattias Petter Johansson):** Excellent for deep dives into JavaScript concepts, often explaining *why* things are the way they are.
    *   **"The Net Ninja":** Covers a wide range of web development topics with clear, concise explanations. Good for seeing patterns.
    *   **"Fireship":** While often fast-paced, their "X in 100 Seconds" videos are great for quickly grasping the core idea of a concept, which you can then deep-dive into.
    *   **"frontend masters" (specifically their free content):** High-quality instructors often explain underlying mechanisms.
*   **For General Programming/Debugging:**
    *   **"Computerphile":** Explores computer science concepts at a foundational level (algorithms, data structures, how computers work). Essential for building strong mental models.
    *   **"Google Chrome Developers" (Debugging series):** Specific tutorials on using browser developer tools for debugging, which are transferable skills.

### **Books (For Deep Fundamentals)**

These books are about building a mindset and understanding principles, not just syntax.

1.  **"Clean Code: A Handbook of Agile Software Craftsmanship" by Robert C. Martin (Uncle Bob):**
    *   **Why:** This is the bible for writing readable, maintainable, and understandable code. It directly addresses the qualities that make code easy to read, debug, and review. It will fundamentally change how you *think* about code.
2.  **"The Pragmatic Programmer: Your Journey To Mastery" by David Thomas and Andrew Hunt:**
    *   **Why:** Focuses on practical advice for software development, including debugging strategies, dealing with errors, and how to approach problems. It's about building good habits and a professional mindset.
3.  **"Code Complete" by Steve McConnell:**
    *   **Why:** A comprehensive guide to software construction. It covers everything from design to debugging to testing, emphasizing clarity and quality. It's dense but incredibly valuable for understanding the *craft* of programming.
4.  **"Debugging: The 9 Indispensable Rules for Finding Even the Most Elusive Software and Hardware Problems" by David J. Agans:**
    *   **Why:** A focused book purely on the art and science of debugging. It provides a systematic approach to problem-solving that can be applied to any code.

Remember, these skills are built through deliberate practice. Dedicate specific time each week to reading unfamiliar code, trying to debug a simulated bug, or actively participating in PRs (even if just silently reviewing others' changes).
