---
title: "The 5 Layers and 5 Concerns of Modern AI"
date: 2026-06-01
draft: false
tags: ["ai", "engineering"]
description: "The modern AI landscape is sprawling and growing (very) fast. Here are the 5 layers and 5 concerns I use every day to make sense of it."
---

I have a lot of discussions about AI these days, in fact, it's almost exclusively what I talk about in a work context. Some months ago, I found myself in a difficult position, in trying to wrangle with all of the different components that I was discussing. There is a rapidly growing and sprawling landscape to try and get your head around and I eventually had to sit down and try and model it in a simple, describable way.

Below I've shared my thinking in two sections: 5 layers and 5 concerns that I use to group these topics into a comprehensive mental model.

## The 5 Layers

The 5 core layers attempts to cover the complete landscape of AI today:

| Layer | Question | Examples |
|---|---|---|
| **Application** | What does the user see and interact with? | Products, integrations, APIs, productivity tools, coding assistants, consumer applications |
| **Orchestration** | How is the model directed and coordinated? | Agents, multi-agent workflows, prompt engineering, LangChain/LangGraph, model routing, hybrid local/remote workflows, multi-model pipelines |
| **Model** | What intelligence is being used and how is it configured? | Model selection, training, fine-tuning, inference serving, self-hosted vs API, local vs remote |
| **Access** | How does the model reach the world beyond its training? | MCP, RAG, knowledge graphs, context graphs, embeddings, tool calls, APIs, memory retrieval |
| **Data** | What information exists and where does it live? | Relational databases, blob stores, vector stores, data lakes, lakehouses, time-series databases, websites, unstructured data |

## The 5 Concerns

The next five are related but separate concerns that a business or adopter of AI should be thinking about.

| Concern | Question | Examples |
|---|---|---|
| **Capability** | What can we do, and why does it matter? | Industry capability frontier, organisational maturity, adoption readiness, use case selection, skills and talent, strategic intent |
| **Economics** | Is it worth doing, and can we afford it? | Cost of inference, infrastructure spend, build vs buy decisions, ROI, investment prioritisation, pricing models |
| **Controls** | How do we protect and constrain it? | Access control, data security, guardrails, misuse prevention, prompt injection, audit trails, cyber threats |
| **Operability** | How do we run and sustain it? | Observability, monitoring, MLOps/LLMOps, scalability, reliability, incident response, testing and evaluation, performance |
| **Governance** | How do we keep ourselves accountable for it? | Ethics, compliance, regulation, responsible AI policy, model risk, bias and fairness, explainability, data governance |

I use these when discussing AI with clients, from strategy to technical implementation. I've refined and tweaked this over the last few months. In upcoming posts, I'll likely refer back to these lists quite a bit. I've found it a very useful tool, I hope you will too.
