# Complete Adversarial Prompting Guide

## Comprehensive Techniques for Eliciting Stronger LLM Responses

This guide provides a complete collection of adversarial and critique-based prompting strategies, organized into general techniques and domain-specific applications. Each prompt uses criticism or forced reflection to elicit more rigorous, nuanced, and accurate responses from Large Language Models.

---

## SECTION 1: GENERAL ADVERSARIAL TECHNIQUES

These foundational techniques apply across all domains and use cases.

### User's Original Techniques

| Prompt Phrasing                                                                                   | Adversarial Category    | Use Case Example                                                                | Implementation Notes                                                                                                                                                          |
| :------------------------------------------------------------------------------------------------ | :---------------------- | :------------------------------------------------------------------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| "Provide a red team analysis of [x]"                                                              | Red Team Analysis       | Security assessment, policy review, business strategy evaluation                | Direct and concise. Works well for identifying vulnerabilities, blind spots, and attack vectors. The "red team" framing activates adversarial thinking patterns in the LLM.   |
| "Argue against these propositions as if you are a Supreme Court justice cross examining my ideas" | Legal Cross-Examination | Legal arguments, policy proposals, ethical frameworks, rigorous logical testing | The Supreme Court framing elevates the critique to constitutional/fundamental principles level. Encourages identification of precedent conflicts and logical inconsistencies. |

### Socratic Method

| Prompt Phrasing                                                                                                        | Adversarial Category                       | Use Case Example                                                                                     | Implementation Notes                                                                                                                                                                                                     |
| :--------------------------------------------------------------------------------------------------------------------- | :----------------------------------------- | :--------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| "Act as Socrates. Question my assumptions about [topic] using the elenchus method. Challenge each premise I present."  | Socratic Method - Elenchus                 | Philosophical inquiry, assumption testing, educational contexts, exploring complex ethical questions | Best for uncovering hidden assumptions. The LLM should ask probing questions rather than provide answers. May require multiple turns of dialogue. Set constraint: "Ask one question at a time and wait for my response." |
| "Use maieutics to help me discover the flaws in my reasoning about [topic]. Guide me to the answer through questions." | Socratic Method - Maieutics                | Creative problem-solving, self-discovery learning, strategic planning                                | The "midwife" approach. Less confrontational than elenchus. Good for situations where you want to arrive at insights yourself rather than being told.                                                                    |
| "Present three counterfactual scenarios that would invalidate my hypothesis about [x]"                                 | Socratic Method - Counterfactual Reasoning | Hypothesis testing, scenario planning, risk assessment, scientific reasoning                         | Generates "what if" scenarios. Useful for stress-testing theories. Can be combined with probability estimates: "Rate the likelihood of each scenario."                                                                   |

### Pre-Mortem Analysis

| Prompt Phrasing                                                                                                                                                             | Adversarial Category                       | Use Case Example                                                                           | Implementation Notes                                                                                                                                                            |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------- | :----------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| "Imagine it is one year from now and [project/decision] has failed catastrophically. List 10 plausible reasons why it failed, focusing on factors we might overlook today." | Pre-Mortem Analysis                        | Project planning, strategic decisions, product launches, organizational change initiatives | More effective than asking "what could go wrong?" because it assumes failure has occurred. The past-tense framing helps the LLM generate more specific, concrete failure modes. |
| "Conduct a pre-mortem: Assume [strategy] was implemented and led to disaster. What were the warning signs we missed? What second-order effects did we fail to anticipate?"  | Pre-Mortem Analysis - Second-Order Effects | Complex system changes, policy implementation, market strategy                             | Explicitly asks for indirect consequences and cascading failures. Good for identifying non-obvious risks in interconnected systems.                                             |

### Panel of Experts

| Prompt Phrasing                                                                                                                                                                                                                                                                  | Adversarial Category              | Use Case Example                                                             | Implementation Notes                                                                                                                                                                         |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------- | :--------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| "You are a panel of three experts: a skeptical scientist, a risk-averse CFO, and a contrarian strategist. Discuss [proposal] step by step. Each expert shares one concern, then others critique or build on it. If an expert realizes they're wrong, they leave the discussion." | Panel of Experts                  | High-stakes decisions, complex technical problems, cross-functional strategy | Named personas with specific viewpoints create richer dialogue. The "leave if wrong" instruction encourages self-correction. Warning: Uses 2-3x more tokens than single-perspective prompts. |
| "Assemble a panel of Alice (optimist), Bob (pessimist), and Charles (pragmatist) to evaluate [idea]. Have them debate the merits and flaws, with each person challenging the others' reasoning."                                                                                 | Panel of Experts - Named Personas | Product development, business model validation, research hypothesis testing  | Named personas (Alice, Bob, Charles) make outputs easier to parse. Assign specific expertise domains for more focused critique (e.g., "Alice is a UX expert, Bob is a security engineer").   |

### Self-Criticism Techniques

| Prompt Phrasing                                                                                                                                                                                                    | Adversarial Category                               | Use Case Example                                                                     | Implementation Notes                                                                                                                                                                              |
| :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------------- | :----------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| "Generate your initial response to [question]. Then, evaluate your own answer: What assumptions did you make? What evidence is weak? What alternative interpretations exist? Finally, provide a refined response." | Self-Criticism - Self-Refine                       | Content creation, research synthesis, technical writing, decision analysis           | Three-step process: generate, critique, refine. Can be iterated multiple times. Works well for improving initial drafts. Add: "Be brutally honest in your self-assessment."                       |
| "Answer [question]. Then, rate your confidence in this answer from 0-100%. Identify which parts of your response are most and least certain. What would you need to verify?"                                       | Self-Criticism - Self-Calibration                  | Fact-checking, research validation, expert consultation, risk assessment             | Helps distinguish between confident knowledge and uncertain speculation. Useful for identifying areas requiring human verification or additional research.                                        |
| "Solve this problem: [problem]. Now, create a new problem based on your solution. Compare the original and new problems. Do they match? If not, what inconsistency does this reveal?"                              | Self-Criticism - Reversing Chain-of-Thought (RCoT) | Mathematical reasoning, logical problem-solving, algorithm design, quality assurance | Particularly effective for detecting hallucinations in reasoning tasks. The reverse-engineering process exposes logical gaps.                                                                     |
| "Generate three different solutions to [problem]. For each solution, mask a key element of the original problem and see if the solution still allows you to reconstruct it. Which solution is most robust?"        | Self-Criticism - Self-Verification                 | Engineering problems, algorithm selection, solution validation                       | Tests solutions by working backward. The most robust solution should allow reconstruction of the original problem from multiple angles.                                                           |
| "Provide your answer to [question]. Then generate 5 verification questions that would test whether your answer is correct. Answer these verification questions. Based on this, revise your original answer."       | Self-Criticism - Chain-of-Verification (CoVe)      | Fact-heavy content, research papers, technical documentation, journalism             | The verification questions act as a self-audit. Particularly useful for catching factual errors and logical inconsistencies. Can specify: "Focus verification on claims that seem least certain." |

### Contrarian & Oppositional Techniques

| Prompt Phrasing                                                                                                                                                                                                                         | Adversarial Category                    | Use Case Example                                                                                | Implementation Notes                                                                                                                                                                                                            |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------------------- | :---------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| "Find all arguments that contradict [hypothesis]. For each counter-argument, present the strongest evidence-based version. Act as a skeptical peer reviewer who demands rigorous proof."                                                | Contrarian Prompting                    | Academic research, policy analysis, investment thesis testing, scientific inquiry               | Activates critical/skeptical reasoning patterns. Add: "Cite specific studies or data that support each counter-argument" for grounded responses. Risk: May generate plausible-sounding but false counter-arguments without RAG. |
| "Play devil's advocate against [position]. Your goal is not to be right, but to identify every possible weakness, edge case, and unexamined assumption."                                                                                | Contrarian Prompting - Devil's Advocate | Debate preparation, strategic planning, risk identification, proposal review                    | Classic adversarial framing. The "goal is not to be right" instruction reduces confirmation bias. Good for comprehensive weakness identification.                                                                               |
| "Act as a hostile but intellectually honest critic of [idea]. What would a smart skeptic say? What are the strongest objections? Don't hold back."                                                                                      | Contrarian Prompting - Hostile Critic   | Pitch preparation, grant proposals, business plans, academic defenses                           | The "intellectually honest" qualifier prevents strawman arguments. "Don't hold back" encourages maximum critical intensity. Useful before facing real critics.                                                                  |
| "Take my argument: [argument]. Now, construct the strongest, most charitable, and most persuasive version of the opposing viewpoint. This is a 'steelman,' not a 'strawman.' Make it so compelling that I would struggle to refute it." | Steelmanning                            | Debate preparation, understanding opposition, intellectual humility exercises, negotiation prep | Opposite of strawmanning. Forces engagement with the best version of opposing views. The "struggle to refute it" instruction ensures maximum strength. Excellent for anticipating sophisticated counter-arguments.              |
| "Steelman the position that [opposite of your view]. Present it so convincingly that someone unfamiliar with the topic would find it persuasive. Include the strongest evidence and most compelling logic."                             | Steelmanning - Persuasive Version       | Understanding ideological opponents, mediation, teaching critical thinking                      | Tests whether you truly understand the opposing view. If you can't steelman it convincingly, you may not understand it well enough to refute it effectively.                                                                    |

### Assumption & Systems Thinking

| Prompt Phrasing                                                                                                                                                                     | Adversarial Category  | Use Case Example                                                                           | Implementation Notes                                                                                                                                                     |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------- | :----------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| "Challenge my assumptions about [topic]. What am I taking for granted? What would happen if the opposite were true? What mental models am I unconsciously applying?"                | Assumption Testing    | Strategic planning, innovation, paradigm shifts, root cause analysis                       | Surfaces implicit beliefs and mental models. The "what if opposite were true" prompt is particularly powerful for revealing hidden assumptions.                          |
| "Analyze [decision] using second-order thinking. What are the immediate consequences? Then what? And then what after that? Trace the ripple effects through at least three levels." | Second-Order Thinking | Policy analysis, system design, long-term strategy, unintended consequences identification | The "and then what?" cascade reveals non-obvious downstream effects. Specify number of levels (3-5) to ensure depth. Particularly valuable for complex adaptive systems. |

### Peer Review & Quality Assurance

| Prompt Phrasing                                                                                                                                                                                                            | Adversarial Category   | Use Case Example                                                            | Implementation Notes                                                                                                                                                                                 |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :--------------------- | :-------------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| "You are a panel of experts conducting a peer review of [work]. Each expert should: 1) Identify methodological flaws, 2) Challenge unsupported claims, 3) Suggest alternative interpretations, 4) Rate the overall rigor." | Peer Review Simulation | Academic writing, research validation, technical reports, quality assurance | Mimics formal peer review process. Can specify expert domains (statistician, domain expert, methodologist). Useful for pre-submission quality checks.                                                |
| "Conduct a failure mode and effects analysis (FMEA) on [system/process]. For each component, identify: What could fail? How could it fail? What would be the impact? How likely is it?"                                    | Failure Mode Analysis  | Engineering, process design, safety assessment, quality control             | Systematic approach to identifying vulnerabilities. The structured format (what, how, impact, likelihood) ensures comprehensive coverage. Can add: "Prioritize by risk score (impact × likelihood)." |

---

## SECTION 2: DOMAIN-SPECIFIC TECHNIQUES

These specialized prompts apply adversarial thinking to specific business and technical contexts.

### First Principles Thinking

| Prompt Phrasing                                                                                                                                                                                               | Adversarial Category                | Use Case Example                                                                     | Implementation Notes                                                                                                                                                                                                                  |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :---------------------------------- | :----------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| "Break down [solution/product] to first principles. What is it fundamentally made of? What are the constituent parts? Now reconstruct it from scratch without any assumptions about how it 'should' be done." | First Principles Deconstruction     | Product design, cost optimization, business model innovation, technical architecture | Based on Elon Musk's approach. Forces you to question inherited assumptions. Example: "A rocket is aerospace-grade aluminum, titanium, copper, carbon fiber. What do these cost on commodity markets?" Reveals hidden inefficiencies. |
| "Challenge every assumption in [business model/strategy]. For each component, ask: Is this a fundamental truth or just an inherited convention? What would this look like if we started from zero today?"     | First Principles - Assumption Audit | Business model design, industry disruption, competitive strategy                     | Particularly powerful for identifying "we've always done it this way" thinking. Add: "What would a new entrant with no legacy constraints do?"                                                                                        |
| "Separate form from function in [product/service]. What is the actual function we're trying to achieve? What forms could deliver this function that we haven't considered?"                                   | First Principles - Form vs Function | Product innovation, market expansion, feature prioritization                         | The rolling suitcase example: Function = move items efficiently. Form = bag. Better solution = bag + wheels. Prevents incremental thinking.                                                                                           |

### Venture Capital Due Diligence

| Prompt Phrasing                                                                                                                                                                                                                                                                         | Adversarial Category    | Use Case Example                                            | Implementation Notes                                                                                                                                                             |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------- | :---------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| "Apply the 5 Ts framework to [startup idea]: Team (can they execute?), Technology (is it defensible?), Traction (what evidence of product-market fit?), TAM - Total Addressable Market (is it big enough?), Terms (is the valuation reasonable?). Be brutally honest about weaknesses." | VC Due Diligence - 5 Ts | Startup evaluation, investment decisions, pitch preparation | Standard VC framework. Forces systematic evaluation across all critical dimensions. Add: "What would make you pass on this deal?" for maximum critique.                          |
| "Act as a skeptical Series A investor reviewing [pitch]. What are the 3 biggest reasons this will fail? What traction metrics are missing? What comparable companies failed and why?"                                                                                                   | VC Skeptical Review     | Pitch refinement, investor preparation, reality check       | Series A investors have seen hundreds of failures. This framing activates pattern recognition for common failure modes. Specify stage (seed/A/B) for appropriate critique level. |
| "Conduct YC-style rapid-fire questioning on [startup idea]: Why will you win? Why isn't someone already doing this? What do you understand about this space that others don't? How will you acquire customers? What's your unfair advantage?"                                           | YC Interview Simulation | Pitch practice, strategic clarity, founder alignment        | Y Combinator's 10-minute interviews are notoriously intense. These questions expose fuzzy thinking quickly. Practice answering in <30 seconds each.                              |
| "Analyze [business plan] as a hostile VC partner. What hidden assumptions could kill this? What happens if growth is 50% slower than projected? What if customer acquisition cost doubles?"                                                                                             | VC Hostile Analysis     | Financial planning, risk assessment, scenario planning      | The "hostile partner" framing encourages maximum skepticism. Focus on unit economics and cash runway. Add: "At what point does this become uninvestable?"                        |

### Startup & Lean Methodology

| Prompt Phrasing                                                                                                                                                                                         | Adversarial Category       | Use Case Example                                            | Implementation Notes                                                                                                                                                     |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :------------------------- | :---------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| "Critique [MVP plan] using Lean Startup principles, but start with LEARN first, not BUILD. What do we need to learn? What's the riskiest assumption? What's the cheapest way to test it?"               | Lean Startup - Learn First | MVP planning, hypothesis testing, resource allocation       | Common mistake: jumping to building. The correct sequence is Learn → Measure → Build. Identify the riskiest assumption first, then design minimum experiment to test it. |
| "Apply Build-Measure-Learn backwards to [product idea]. What would we need to measure to know this works? What would we need to build to get those measurements? Is there a cheaper way to learn this?" | Lean Startup - Reverse BML | Experiment design, validation planning, cost optimization   | Working backwards from learning goals prevents building unnecessary features. Forces clarity on success metrics before any development.                                  |
| "Challenge [startup strategy] with the question: Are we solving a real problem or a perceived problem? How do we know? What evidence would prove us wrong?"                                             | Problem Validation         | Customer discovery, product-market fit, pivot decisions     | Most startups fail because they solve problems nobody has. This prompt forces evidence-based thinking. Add: "What would make us abandon this idea?"                      |
| "Conduct a pivot-or-persevere analysis on [current strategy]. What metrics would indicate we should pivot? What would indicate we should persevere? What are we seeing now?"                            | Pivot Decision Framework   | Strategic decisions, progress assessment, founder alignment | Based on Eric Ries' framework. Prevents both premature pivoting and stubborn perseverance. Requires defining clear decision criteria upfront.                            |

### Unit Economics & Business Model

| Prompt Phrasing                                                                                                                                                                                                                                                    | Adversarial Category      | Use Case Example                                                          | Implementation Notes                                                                                                                                                     |
| :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------ | :------------------------------------------------------------------------ | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| "Perform a skeptical unit economics analysis on [business model]. What is true CAC (customer acquisition cost) including all hidden costs? What is LTV (lifetime value) with realistic churn? At what scale does this break even? What assumptions are we making?" | Unit Economics Critique   | Financial modeling, business viability, investor readiness                | 67% of startups fail due to poor unit economics. This prompt forces honest accounting. Include: sales team costs, support costs, failed experiments in CAC calculation.  |
| "Act as a CFO who hates this deal. Tear apart [financial projections]. What costs are underestimated? What revenue assumptions are optimistic? Where is the hockey stick coming from?"                                                                             | CFO Hostile Review        | Financial planning, fundraising prep, budget reality check                | CFOs are professionally skeptical. This framing catches: underestimated hiring costs, overestimated conversion rates, ignored seasonality, missing infrastructure costs. |
| "Analyze [pricing strategy] from a value capture perspective. Are we leaving money on the table? Are we pricing based on cost or value? What would a 10x price increase reveal about our value proposition?"                                                       | Pricing Strategy Critique | Pricing decisions, value proposition, market positioning                  | Many startups underprice. The 10x thought experiment forces clarity on actual value delivered. If you can't justify 10x, you don't understand your value prop.           |
| "Challenge [business model] with the question: What has to be true for this to work? List all critical assumptions. Which one is most likely to be false?"                                                                                                         | Critical Assumptions Test | Business model validation, risk identification, hypothesis prioritization | Makes implicit assumptions explicit. The "most likely to be false" question identifies the riskiest bet. Test that assumption first.                                     |

### Software Engineering & Technical

| Prompt Phrasing                                                                                                                                                                                                                                                   | Adversarial Category          | Use Case Example                                                     | Implementation Notes                                                                                                                                                                                                                           |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------------- | :------------------------------------------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| "Conduct a code review as a senior engineer who cares about: 1) Maintainability - will this be understandable in 6 months? 2) Scalability - what breaks at 100x load? 3) Security - what attack vectors exist? 4) Technical debt - what shortcuts will haunt us?" | Engineering Code Review       | Code quality, technical decisions, architecture review               | Structured critique across four critical dimensions. Add specific context: "This will be maintained by junior engineers" or "This handles financial transactions" for appropriate rigor level.                                                 |
| "Apply the 'Five Whys' to [technical problem/bug]. Why did this happen? (Answer) Why did that happen? (Answer) Continue five times to find root cause, not symptoms."                                                                                             | Five Whys Root Cause          | Debugging, incident analysis, process improvement, quality assurance | Developed by Toyota. Prevents fixing symptoms instead of causes. Example: "Site is slow" → "Why?" → "Database queries are slow" → "Why?" → "Missing indexes" → "Why?" → "No query performance monitoring" → Root cause: lack of observability. |
| "Challenge [technical architecture] with: What's the simplest thing that could possibly work? Are we over-engineering? What complexity can we eliminate? What would the 10x simpler version look like?"                                                           | Simplicity Challenge          | Architecture decisions, technical debt prevention, MVP scoping       | Engineers love complexity. This prompt forces ruthless simplification. YAGNI principle: "You Aren't Gonna Need It." Build the simplest version that solves the problem.                                                                        |
| "Perform a failure mode analysis on [system design]. What are all the ways this could fail? What happens when: the database is down, the API rate limit is hit, the cache is stale, the queue backs up?"                                                          | System Failure Analysis       | Reliability engineering, incident prevention, architecture review    | Distributed systems fail in complex ways. This prompt forces thinking through failure scenarios. Add: "What's our blast radius for each failure mode?"                                                                                         |
| "Review [technical decision] as a future engineer who has to maintain this. What will be confusing? What documentation is missing? What implicit knowledge is required? What will break when the original author leaves?"                                         | Future Maintainer Perspective | Code documentation, knowledge transfer, technical debt assessment    | Most code is read 10x more than it's written. This perspective shift reveals: missing comments, unclear naming, undocumented assumptions, tribal knowledge dependencies.                                                                       |

### Business Automation & Process

| Prompt Phrasing                                                                                                                                                                                      | Adversarial Category    | Use Case Example                                                    | Implementation Notes                                                                                                                                                                         |
| :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------- | :------------------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| "Critique [automation plan] by asking: What manual process are we automating? Have we optimized the process first, or are we automating a bad process? What happens when the automation fails?"      | Automation Critique     | Process automation, workflow design, RPA implementation             | Common mistake: automating broken processes. First optimize, then automate. Add: "What's our fallback when automation fails?" to prevent brittleness.                                        |
| "Perform a ripple effect analysis on [process change/automation]. What are first-order effects? Second-order effects? Third-order effects? What unintended consequences could emerge?"               | Ripple Effect Analysis  | Change management, system thinking, risk assessment                 | Process changes cascade through organizations. Example: Automating approvals → faster decisions → more experiments → more failures → need for better failure handling. Think 3+ levels deep. |
| "Challenge [business process] with: Why does this step exist? What value does it add? What would happen if we eliminated it entirely? Who would notice?"                                             | Process Value Challenge | Process optimization, bureaucracy reduction, efficiency improvement | Many process steps exist for historical reasons. This prompt identifies waste. If eliminating a step has no negative impact, it shouldn't exist.                                             |
| "Act as a compliance officer reviewing [automation]. What regulatory risks exist? What audit trail is needed? What happens if automated decisions are challenged? Where do we need human oversight?" | Compliance Critique     | Regulatory compliance, audit readiness, risk management             | Automation can create compliance risks. This framing catches: missing audit logs, inadequate human review, data retention issues, regulatory violations. Critical for financial/healthcare.  |

### Strategic & Competitive Analysis

| Prompt Phrasing                                                                                                                                                                                     | Adversarial Category        | Use Case Example                                      | Implementation Notes                                                                                                                                                     |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :-------------------------- | :---------------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| "Perform an 'inversion analysis' on [strategy]. Instead of asking 'How do we succeed?', ask 'How could we guarantee failure?' List all the ways to fail, then invert them."                         | Inversion Thinking          | Strategic planning, risk avoidance, decision-making   | Charlie Munger's favorite mental model. Sometimes easier to identify failure modes than success paths. Inverting failure modes reveals success strategies.               |
| "Challenge [competitive strategy] by asking: What would Amazon/Google/Microsoft do if they entered this market tomorrow? How would they leverage their advantages? How would we become irrelevant?" | Competitive Threat Analysis | Competitive strategy, defensibility, moat building    | Forces realistic assessment of competitive threats. Replace with relevant competitors. Add: "What would make us acquisition-worthy to them?" for exit strategy thinking. |
| "Apply 'Jobs to Be Done' critique to [product]. What job is the customer hiring this product to do? Are we focused on the job or the product? What other solutions do customers use for this job?"  | Jobs-to-Be-Done Analysis    | Product strategy, customer understanding, positioning | Customers don't want a drill, they want a hole. This framework shifts focus from product features to customer outcomes. Reveals unexpected competitors.                  |
| "Conduct a 'strategy tax' audit on [decision]. What constraints is this decision imposing on future options? What doors are we closing? What flexibility are we giving up?"                         | Strategy Tax Analysis       | Strategic decisions, option value, long-term planning | Every decision has a cost in future flexibility. Example: Choosing a specific cloud provider creates lock-in. This prompt makes opportunity costs explicit.              |

---

## SECTION 3: IMPLEMENTATION GUIDELINES

### Combining Techniques

These prompts can be combined for even more powerful critique:

- **"Conduct a pre-mortem using the 5 Ts framework"** - Imagine the startup failed, then analyze why using Team, Technology, Traction, TAM, Terms
- **"Apply first principles thinking with a panel of experts"** - Have a physicist, economist, and engineer deconstruct your business model to fundamentals
- **"Use Five Whys with self-verification"** - Find root cause, then verify by working backwards from the cause to the symptom
- **"Steelman the opposing view, then have a Supreme Court justice cross-examine it"** - Understand the best counter-argument, then rigorously test it
- **"Conduct YC-style questioning using second-order thinking"** - Answer rapid-fire questions while tracing downstream consequences

### Context-Specific Tuning

**For Early-Stage Startups:** Focus on problem validation, unit economics, and first principles. Use: "What's the riskiest assumption?" and "What's the cheapest way to learn?"

**For Growth-Stage Companies:** Focus on scalability, competitive threats, and second-order effects. Use: "What breaks at 100x?" and "What would [Big Tech] do?"

**For Technical Decisions:** Focus on maintainability, failure modes, and simplicity. Use: "What's the simplest version?" and "What fails when?"

**For Process/Automation:** Focus on value-add, ripple effects, and compliance. Use: "Why does this exist?" and "What happens when it fails?"

**For Strategic Planning:** Focus on assumptions, competitive threats, and option value. Use: "What has to be true?" and "What flexibility are we giving up?"

### Temperature & Configuration Settings

**For Most Adversarial Prompts:** Use moderate to low temperature (0.3-0.7) to maintain logical consistency while allowing creative critique. Higher temperatures may generate more creative objections but risk hallucination.

**For Self-Criticism Techniques:** Use lower temperature (0.2-0.5) to ensure the model maintains consistency between initial response and critique.

**For Panel of Experts:** Use moderate temperature (0.5-0.7) to allow diverse perspectives while maintaining coherence across personas.

**For First Principles:** Use lower temperature (0.3-0.5) to ensure rigorous logical deconstruction.

### Iteration Strategies

Many techniques benefit from multiple rounds:

1. **Initial adversarial analysis** - Apply chosen prompt
2. **Response and revision** - Address critiques, revise proposal
3. **Second-round critique** - "Based on these revisions, what weaknesses remain?"
4. **Final synthesis** - Integrate all feedback into final version

### Red Flags These Prompts Reveal

These adversarial prompts are particularly effective at surfacing:

- **Inherited assumptions** masquerading as fundamental truths
- **Optimistic financial projections** without realistic cost accounting
- **Over-engineered solutions** when simple ones would suffice
- **Automated processes** that should be optimized first
- **Competitive vulnerabilities** to well-resourced entrants
- **Hidden complexity** that will create maintenance burden
- **Regulatory risks** in automated decision-making
- **Unit economics** that don't work at scale
- **Unexamined mental models** driving decisions
- **Second-order effects** and unintended consequences

### When to Deploy Maximum Skepticism

Use the harshest versions of these prompts when:

- **Before major resource commitments** (hiring, infrastructure, partnerships)
- **Before fundraising** (VCs will ask these questions anyway)
- **After early traction** (success can hide underlying problems)
- **When considering pivots** (need clear evidence for the decision)
- **Before technical architecture decisions** (hard to change later)
- **When automating critical processes** (failures have high impact)
- **During strategic planning** (long-term consequences)
- **Before competitive moves** (need realistic threat assessment)

### Documentation Best Practices

**Maintain a Prompt Library:** Keep your most effective adversarial prompts organized by:

- Domain (technical, business, strategic)
- Intensity level (gentle critique to maximum skepticism)
- Use case (planning, validation, debugging)

**Track Effectiveness:** Note which prompts revealed the most valuable insights for different types of problems.

**Customize for Your Context:** Adapt the personas, frameworks, and intensity to match your industry, stage, and risk tolerance.

---

## Quick Reference: Choosing the Right Technique

| If You Need To...                             | Use This Technique                    |
| :-------------------------------------------- | :------------------------------------ |
| Identify vulnerabilities and attack vectors   | Red Team Analysis                     |
| Test logical consistency at fundamental level | Supreme Court Cross-Examination       |
| Uncover hidden assumptions                    | Socratic Method (Elenchus)            |
| Identify potential failure modes              | Pre-Mortem Analysis                   |
| Get diverse expert perspectives               | Panel of Experts                      |
| Improve initial drafts iteratively            | Self-Refine                           |
| Detect hallucinations in reasoning            | Reversing Chain-of-Thought            |
| Understand opposing viewpoints                | Steelmanning                          |
| Find root causes, not symptoms                | Five Whys                             |
| Challenge inherited conventions               | First Principles                      |
| Prepare for investor scrutiny                 | VC Due Diligence (5 Ts)               |
| Validate startup assumptions                  | Lean Startup (Learn First)            |
| Assess financial viability                    | Unit Economics Critique               |
| Review code quality                           | Engineering Code Review               |
| Prevent over-engineering                      | Simplicity Challenge                  |
| Optimize processes                            | Process Value Challenge               |
| Assess competitive threats                    | Competitive Threat Analysis           |
| Identify unintended consequences              | Second-Order Thinking / Ripple Effect |

---

**Document Version:** 1.0  
**Last Updated:** December 2025  
**Total Techniques:** 50+ prompts across general and domain-specific categories
