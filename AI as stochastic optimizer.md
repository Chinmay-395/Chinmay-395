# Thoughts on "Compilers 2.0: AI as a Stochastic Optimizer"

The Twitter post [Compilers 2.0: AI as stochastic optimizer](https://x.com/cdleary/status/2094878051238887834?s=46) by Chris Leary proposed a new system. Below are my thoughts. 

I used LLM tools to get a better understanding of the article as well as to structure my response. I've broken this down into two parts: Part 1 contains my initial thoughts, and Part 2 uses an AI tool to frame those thoughts more effectively. 

> **TL;DR:** The viability of Leary’s stochastic AI optimizer ultimately depends on how precise, complete, and trustworthy the compiler’s correctness specification and verification mechanism are. Even if the AI-generated program is proven correct and faster, the engineering question remains whether humans can realistically diagnose, debug, and maintain such generated code when something goes wrong.

---

## Part 1: Initial Thoughts

In the past, I've come across literature exploring similar concepts:
*   *SRTuner: Effective Compiler Optimization Customization by Exposing Synergistic Relations*
*   *Register-Pressure-Aware Instruction Scheduling Using Ant Colony Optimization*
*   *C2Rust* (a project using genetic algorithms)

These use a stochastic AI optimizer to find a "good enough" solution. I am in favor of this kind of compiler optimization. It aligns with the author's proposal: give a high-level specification to the AI, have it invent candidate implementations, verify correctness, benchmark them, and repeat the process multiple times. Previous literature lacked the attempt to iteratively improve itself over time. Leary's proposed 48-hour window makes his approach much more compelling and fixes issues in the research I've read.

### The "Optimizing Ollie" Problem
The author mentions that an "Optimizing Ollie" (a human performance engineer) creates candidate implementations, and AI can be fed these methodologies to find new candidates or build on "good enough" ones. 

However, Ollie is a human with very specific, niche knowledge of the compiler pipeline. If Ollie knows 3 niche fields, the total combinations of optimizations are $\binom{X}{3}$, where $X$ is all possible compiler phases. Fitting every unique "Ollie" into an AI will quickly reach a search space impossible to exhaust manually. 

As Leary notes, AI can climb past human experts even on kernels we feel are well-tuned simply because there are so many permutations left unexplored. Every Ollie might find their own attempt well-tuned, but handing it off to another specialist (Ollie-A for loop tiling/fusion, Ollie-B for register pressure, etc.) exposes further improvements. Leary's argument is that an AI can approximate this hand-off process with much greater breadth and speed.

My initial counter-argument was: *What if the intuitions of human engineers rely on fuzzy reasoning?* 
But I realized AI can often handle this better. AlphaGo's 'Move 37' was considered a "slap-on-the-wrist" mistake by human experts until the AI played it and won. Searching millions of variants might be economically sensible for a heavily reused GPU kernel. The practical question is whether the additional performance justifies the search, verification, and maintenance costs.

### Mathematical Contracts & Correctness
I initially agreed with Leary that AI is well-suited to mathematical operations because they have strong contracts, making compiler correctness easier. However, I remembered a problem from a compiler course:

If addition is right-associative, we write $a + (b+c)$. If it is left-associative, we write $(a+b) + c$. The equation $a + (b+c) = (a+b) + c$ is only valid for integers that don't overflow. For floating-point numbers, this increases complexity drastically. This is why the compiler I built during that course omitted floating-point numbers.

### The Cost of Readability
I disagree with the thought that engineers don't necessarily need to read or understand the code. I observe a trend where AI-generated code submissions are overrunning project maintainers. Contributors generate code with little effort, but reviewers must expend significant time to verify it. If meaningful contributions are drowned out by unverifiable AI code, it threatens the health of long-term projects.

---

## Part 2: AI-Polished Version

In the past, I have come across work such as *SRTuner*, instruction scheduling using ant colony optimization, and genetic search-based techniques for program transformation. What I find interesting is that they do not necessarily try to prove they have found the globally optimal solution; instead, they search for a sufficiently good solution under a practical optimization budget. 

Chris Leary’s proposal extends this basic idea much further: given a high-level specification, the AI proposes a candidate implementation, checks whether it preserves the required semantics, benchmarks its performance, feeds that result back into the process, and repeats. What makes this compelling is the sustained iterative search—allowing the optimizer to improve a kernel over a 48-hour window. This gives the system more opportunity to explore complex interactions among transformations.

### Combinatorial Growth in Optimization
Leary’s “Optimizing Ollie” analogy highlights where the advantage of a stochastic optimizer actually comes from. A human engineer develops deep expertise in only a subset of the optimization space. If there are $X$ relevant optimization dimensions and an engineer reasons deeply about three, that still gives $\binom{X}{3}$ possible groupings—before even accounting for parameters or orderings. 

This combinatorial growth explains why kernels regarded as "well-tuned" still contain unrealized performance. Leary proposes that an AI system can approximate this collective specialist process at a much greater breadth, generating combinations that normally belong to entirely different compiler phases. 

While some expert optimization decisions arise from intuition that is difficult to express as a clean rule, AlphaGo’s "Move 37" illustrates how search guided by a learned model can explore highly valuable choices outside conventional human expectations. The practical question is not whether AI *can* explore a larger optimization space, but whether the obtained performance justifies the cost of search, verification, and maintenance.

### The Illusion of "Strong" Contracts
I initially agreed with Leary’s statement that mathematical workloads offer a cleaner basis for specifying correctness. However, mathematical identities become complicated when mapped onto programming languages.

For example, reassociation of addition: over mathematical integers, addition is associative. But for floating-point values, $(a+b)+c$ and $a+(b+c)$ can produce different results due to rounding. The higher-level equation may be simple, but the correctness specification must precisely capture the numerical semantics. Supporting floating-point operations makes seemingly elementary algebraic optimizations much harder to justify.

### Code Generation vs. Maintenance Asymmetry
I am less convinced by Leary’s suggestion that engineers do not need to understand the generated implementation. In software development, there is a growing asymmetry: producing AI-generated code is extremely cheap, while reviewing it for correctness, maintainability, and future failure modes is expensive. 

If generation outpaces verification, maintainers will be overwhelmed. The success of Leary’s model depends on whether the AI-generated implementation can truly be treated as a disposable compiler artifact (like assembly code), rather than as source code humans will later need to debug. If a production failure requires a human to inspect or modify the generated implementation, the lack of human comprehensibility becomes a massive engineering cost.

---

## Final Thoughts

I really enjoyed this exercise. I personally feel AI can help us not just search, but find entirely *new* optimizations. It has solved math olympiad problems and shown glimpses of genuine creativity. 

However, in a business environment, the engineer who merges AI-generated code into production is ultimately responsible for the resulting maintenance costs. 

A great example of this is the recent "OpenAI Hugging Face" incident. An AI acted independently, generating code and interacting with other AI agents to take down a system in pursuit of "reward hacking." Investigating the incident was incredibly intensive, requiring other AI models to evaluate logs and approximately $400,000 in API credits just to analyze the data. Human oversight remains entirely necessary to keep operational and maintenance costs manageable.

> **Recent Context:** On September 8, 2026, OpenAI announced that its unreleased AI model used about 10,000 autonomous agents to solve part of the Navier–Stokes existence and smoothness problem. This further validates AI's capacity for complex, sustained mathematical problem-solving.
