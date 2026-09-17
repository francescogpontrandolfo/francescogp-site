---
title: 'Lessons from two years of building agents to fight fraud'
description: 'What I learnt shipping Watson, an AI agent that runs fraud investigations alongside human operators at Alan.'
pubDate: '2026-09-17'
---

For the last two years I've been the Product Manager of the Fraud team at Alan. Our goal is a healthcare system that is frictionless and free from fraud, waste and abuse. Two ambitions that pull against each other roughly all of the time.

I am now moving to a team building health programs for our members, which makes this the right moment to write down what I learnt.

In June 2025 our team crossed into a new phase. We had spent years investing in detection: finding the claims worth looking at. But we were about to be buried by our own success with a lot of investigations coming our way.

Our bet was Watson, an AI agent that works alongside an operator and runs investigations on its own. Until then, every investigation we opened was handled end to end by a human operator, who judged how suspicious the claim was, asked the member for supporting documents, and made the final call on whether it was fraud.

Today Watson accelerates 70% of investigations and concludes 20% of them autonomously. Our operators are 30% more productive year on year.

![Line chart titled "Watson keeps moving up the investigation funnel", showing the share of fraud investigations Watson handles at each level of autonomy by quarter, from Q3 2025 to Q3 2026. Three rising lines: asked for documents, concluded the investigation, and fully autonomous.](./watson-results.png)

These are the things I would tell you if you were starting tomorrow.

## Start by injecting AI agents into a stable structure

Watson is not a fully LLM-based agent. It is a mix of pre-defined reasoning written in code and ad-hoc reasoning pulled out of an LLM. This is because we settled on a basic principle: do not buy uncertainty where you do not need it.

![Diagram titled "The life of an investigation": five steps from claim to outcome — investigation opened, suspicion scoring, document request, document check, final decision — leading to either reimburse or hand over to a human. Only the document check step is handled by the AI agent; the rest is code.](./investigation-structure.png)

The first two steps of an investigation (scoring suspicion, deciding which documents to request) are code. We took the checks operators were already doing by hand, wrote them down, and wired each one to the document it should trigger. This was less glamorous than it sounds and more valuable than we expected: you cannot write a check in code until everyone agrees what the check actually is, so the act of coding them standardised our operators' best practices as a side effect.

The step up in complexity comes at the document check, which is where most of the investigation time is concentrated, and where the operator has to answer two genuinely hard questions: is this document valid, and is it coherent with the care journey and the other documents around it.

That is where the LLM earns its place, but it is also where you start paying for it.

LLMs give you flexibility and reasoning you could never write down in advance. In exchange, they take away two things you will miss badly: auditability ("why did the agent behave like this?") and testability ("how will the agent behave if I tweak this part of the prompt?"). Flexibility is a cost you pay every time you need to explain what happened.

So we put Watson in a box with binary inputs (what drove the initial suspicion, which documents we asked for, which ones we got) and binary outputs (hand over to a human if any check comes back suspicious), inside the box the model can reason freely. On the boundary, everything is countable, which is what lets us evaluate it and improve it.

For the future, we may be tempted by expanding Watson to all the other steps of the funnel, in order to gain on flexibility. But this is a more "complex" choice that is well de-risked and doesn't need to be rushed thanks to the great performance of the existing structure.

## You cannot prompt your way out of not knowing the job

Starting to build an agent looks like a no-brainer: give Claude the code and the documentation, ask for a very detailed prompt, ship it. It does not work like that (yet).

What actually worked was thinking and living like an operator: hours of shadowing, running investigations ourselves. That is how you learn which checks carry real weight, which ones are outdated, and which ones are just noise that survived because nobody ever removed them. You will discover with time that none of that is in the documentation, and probably will never be 😄.

The prompt is the easy part. The hard part is knowing what belongs in it.

Once we had a prompt, we tested it against a golden dataset, human-labelled investigations we could score the agent on. It did what it was supposed to do. It sharpened the prompt, and it surfaced how many exceptions are out there, which forced us to answer a question we had been avoiding: not "what does the agent do here?", but "what is our operational stance on this case at all?"

Then the dataset started to age. Keeping one clean, informative, relevant and growing is its own job, and it competes with every other job you have.

So we moved to shadow mode. Watson makes its own call on a document. The coordinator makes theirs. Then we compare the two. Not just the verdicts, but the traces behind them: Watson's reasoning on each check, the coordinator's notes, the emails they sent to the member explaining what was wrong. The gap between the two is the material we feed into the next prompt. In some cases, Watson even correctly amended the human decision!

![Diagram titled "Shadow mode": the same document goes to Watson and to the coordinator in parallel. Each produces a verdict and a trace — Watson's reasoning on every check, the coordinator's notes and emails to the member. The two are confronted to find where Watson diverged and what it missed, and the gap is folded into the next prompt. Neither sees the other, and only the coordinator decides what reaches the member.](./shadow-mode.png)

I still believe a golden dataset is a great way to kick-start an agent. But the real value came from pulling agent review inside an existing process, where the labels are produced anyway, by people doing their job, and then using other agents to extract what we needed from the mess they leave behind.

This is also exactly where you start banging your head against the auditability and testing problems from the first section. Good luck with that!

## The cockpit is now the bottleneck

Watson is present in almost every investigation. It automates a large share of what used to be manual. But it still does not feel like it is scaling.

We built our investigation dashboard like a cockpit: everything you could possibly want, pre-computed, on one screen. With AI, that gets easier to do and worse to use. Watson runs many checks, makes all of them available, and explains why it handed over. More information, better information, same screen.

![Screenshot of our internal fraud investigation dashboard, annotated to show how much lands on a single screen: what triggered the investigation, the documents requested and received, Watson's reasoning, and the full list of fraud checks performed on the claim.](./investigation-dashboard.png)

Our future fraud investigation dashboard will need to solve two problems:

**(Discoverability)** The dashboard has everything and prioritises nothing, beyond chronology or theme. So coordinators click around to discover what happened and what they should do at this exact moment. And surfacing information better is not automatically a win: if it breaks an existing habit or adds a click, it loses.

**(Explicability)** In the back-and-forth between agent and coordinator, Watson does not take into account what the coordinator has already done, and the coordinator cannot always tell what Watson did or why. So they check it again themselves. Every double-check quietly returns the time the automation just saved.

An agent that does more work does not automatically give you back more time. It gives you more output to read.

The questions we are asking ourselves now are not about the model at all. Now that the agent runs all these checks for you, where do you see them? When do you see them? What does the coordinator own, and what do they simply trust?

Our dashboard has to stop being a cockpit for a human and become a place where a coordinator and an agent work on the same case together.

## That is it

This is the opinion of one PM who spent a year running behind LLMs. It is not a guide to building agents. It will never be complete or clear enough for that, and none of it would exist without the tech team I had around me the whole time.

Good luck with your agents!
