# MATACS Associate Software Developer Assessment
- **Candidate Name:** Muhammad Amir Jawed
- **Assessment Reference:** MATACS-TH-2026-02
- **Time Actually Spent:** About 60 minutes
- **Repository Link:** [matacs-th-2026-02](https://github.com/amir664/matacs-th-2026-02)

## Repository Files
- [WRITEUP.md](./WRITEUP.md) - Answers to the five design questions.
- [PROMPTS.md](./PROMPTS.md) - AI extraction and scoring prompts.
- [ranking.json](./ranking.json) - Expected ranking of proposals A-E.
- [diagram/](./diagram/) - Process-flow diagram from upload to ranked output.

## Use of AI Tools
I utilized AI tools (such as ChatGPT/Claude) to structure the design documentation, refine the JSON schema for proposal extraction, and formulate the prompt-injection guardrails. 



Initially, the Ai model suggested letting the LLM compute the final weighted scores and rankings directly. I disagreed and corrected this because LLMs can hallucinate or make mathematical errors. I restructured the design so that the Ai strictly handles unstructured text extraction, while all scoring, weight calculations, and pass/fail logic are executed deterministically through code.