AI Ethics
=========

## Links
- [Northern Frontier: In conversation with Abhishek Gupta - Start at 1:20][1]
- [Research Summary: Beyond Near-term and Long-term in AI Etchics Research Priorities][2]
- [Research Summary: The Toxic Potential of YouTube's Feedback Loop][3]
- [A 16-year old AI developer's critical take on AI ethics][4] (try to get to an understanding of AI ethics so your thoughts and questions are more nuanced than this article)

[1]: https://youtu.be/Z3Tme0WU5D8?t=80
[2]: https://montrealethics.ai/research-summary-beyond-near-and-long-term-towards-a-clearer-account-of-research-priorities-in-ai-ethics-and-society/
[3]: https://montrealethics.ai/the-toxic-potential-of-youtubes-feedback-loop/
[4]: https://montrealethics.ai/a-16-year-old-ai-developers-critical-take-on-ai-ethics/


## Areas of AI Ethics
### Bias
- Bias is problematic when it disproportionately affects the outcomes of a certain group. COMPAS criminal recidivism risk assessment program in the US was biased against African Americans.
- There are areas where bias is okay. EX: It's okay if your ML model for diagnosing cardiac diseases has a biass towards having a higher incidence rate for men than women.
- Recognition rates for minority groups should be on par with rates for the majority group.
- Real world data distributions are often different from our "toy" training sets

### Transparency / Explainability
- Useful to find bias, and to increase confidence in decisions made by ML systems

### Accountability
- Developers of ML models should be accountable for the outcomes of their systems.

### Interoperability
- Interoperability is the ability of two or more systems to exchange information
- Usually involves standardizing the format of information being exchanged

### Social Impacts
- General catch-all non-technical category

## Misc
### Goal of Montreal AI Ethics Institute (MAIEI)
- Move from talk to action with AI ethics
- How to run applied experiements
- What are the concrete steps, technical tools, and frameworks that we can build that actually pushes forward research in the field

### Challenges for AI Ethics
- Each community is different. There isn't a clear consensus on what values and morals for a specific are
- Most ML systems optimize for a single performance metric.
- AI ethics research often isn't seen as "technical" enough to deserve respect from the broader ML community

## Questions
- Let's say I'm building a credit scoring model, and I find that one of the most important features is their race. What should I do? Removing that feature from training and inference might lower the performance of my model. Should I construct the training set so that the prior probability distibution is uniform across different racial groups?
    - Abi: Don't view this as a tradeoff, but as a way to gain market differentiation. Large companies serve to gain more from positive PR than marginal gains in unethical AI systems.
    - Thought: Could you build a platform that automatically certifies algorithms as ethical, in an externally verifiable way?

- If there's a tradeoff between ethics and profitability, is there a way to make large corperations prioritize ethics? It seems like a public goods problem; we'd all prefer if certain algorithmic processes didn't take into account demographic information, or optimize for predatory metrics like user engagement, but individual companies might have incentives to do so. Are third parties such as governments, regulations, or cross-company policy groups required to enforce coordination?
    - Abi: YouTube and other recommendation algos that optimize for engagement likely won't change until there's public outcry. Monetization mechanism needs to be decouple from the content distribution mechanism. How do you measure quality of time spent on platform?

- What "low-hanging" areas of open issues in AI ethics are there that somebody with ~15 hours a week could contribute to? Anything that's more technical and concrete than working with a researcher to create policy recommendations?
    - Abi: Contribute to open source. Look at OpenMind. Would intro me to Andrew at OpenMined. Look at White Noise, Microsoft's tool for differential privacy. Fair Learn (Github), Facets (Google - ML visualization), TensorBoard What If (Google).

Question from Abi to me: How can we raise awareness of AI ethics "stuff" among people your age?
- To raise awareness of AI ethics among general population Netflix / YouTube content that casually mentions ethical AI things would be great. E.g. Black mirror style AI issue, but some AI ethics tool or concept is used to actually fix the problem.
- To raise awareness and priority of AI ethics among young AI devs, include ethical AI tools (TensorBoard What If, Facets etc) in the standard workflow. If you can get TensorFlow / PyTorch beginner tutorials to include a subsection on their dog / cat classifier to have a biased dataset check, that'd be a large step forward. Many people think of AI ethics as being too abstract and not technical enough, and this would change that perception.
