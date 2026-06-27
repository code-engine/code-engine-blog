---
title: "The Dune Test"
date: 2026-06-03
draft: false
tags: ["ai", "llm"]
description: "LLMs don't lie. They hallucinate. Understanding this will help you use them better and design your systems for greater accuracy."
---

With the proliferation and widespread adoption of LLMs, I've heard a lot of people talking about how they "lie" recently. I'll admit, I've felt the urge to reach for that word myself, but it's not quite right for what's actually happening. LLMs don't lie as such: the more accurate term is _hallucination_.

LLMs are built on what is known as a corpus, which is all of the data used to train the model. If the information you're looking for isn't in the corpus, then the LLM can't by itself give you the answer. What tends to happen in modern frontier systems is that the orchestration layer and the model will work together to try and fetch the missing information from the internet or any other storage the system is aware of. If that fails, the LLM will do one of two things:

1. Tell you it doesn't know.
2. More likely, hallucinate.

Frontier models and their associated orchestration systems are getting better at the former, but there is still a long way to go. The key thing to understand is that an LLM cannot reason about the knowledge it has or does not have. It is a probabilistic system that takes natural language as an input and produces a result as an output. When  you search for something on Google, if it's not found, you can reason about that yourself, whereas an LLM will essentially answer, with confidence, most of the time.

This is critical to understand, especially if you are building applications and systems on top of an LLM, because if you're not careful, the system can produce incorrect / erroneous results, with confidence.

---

As a concrete example of this, a while back, I wanted to find a specific (and apt) quote from Frank Herbert's Dune. The quote is:

> _“Once men turned their thinking over to machines in the hope that this would set them free. But that only permitted other men with machines to enslave them.”_ — Frank Herbert, Dune

I was fairly certain the quote was in the book, I just couldn't remember where and the exact wording. I thought to myself that this would be a trivial job for an LLM.

I tried several of the frontier models, but not one of them could find it. More interestingly, they all had a go at it and gave very confident, but very wrong answers. One claimed it was in a section that it wasn't, one said it wasn't in Dune at all and that it was in fact in fan fiction designed to sound like Herbert, another said it was in an appendix, another said it was in some versions but not others, and finally another said it was in a different book series altogether.

None of this is true, the quote is in fact in the first Dune book. I found it in seconds on my ebook reader using a basic text search.

So what happened here? How could such sophisticated machinery be outdone by a basic keyword search?

---

The models didn't lie, they did exactly what they were built to do, which is generate the most plausible answer they could from the data they had available.

Dune is not freely available on the open internet. The quote was not widely cited with the applicable chapter and verse online. I can assume with a high degree of certainty that it doesn't exist in any of their training corpora in a way that would allow the model to find it. So each generated the most plausible sounding answer it could construct. And, because each model was trained and tuned on different data and parameters, each produced a slightly different answer.

This example isn't unique either. Ask an LLM about recent sporting events, a breaking news story, or the internal data within your organisation and you will hit the same problem. The information will either postdate the training corpus, or doesn't exist anywhere that the model can reach. Unfortunately, the failure mode is the same for each case, which is a confidently stated, fabricated answer.

---

What this example hopefully illustrates is that there are broadly three classes of question you can ask an LLM:

1. Questions whose answers are well represented in training data
2. Questions whose answers are not in the training data, but are retrievable from external sources
3. Questions whose answers are not in the training data and are unretrievable

The model is highly likely to respond to all three with the same tone of confidence. They have no reliable way of differentiating which class it is responding to and this is just a characteristic of how these systems work.

It's worth noting that the better frontier models are actively trying to improve this. Some will have a confidence score associated with their responses and will sometimes be able to tell you when they are not sure, flag that the information might be outdated or decline the answer rather than confabulate. But it's far from a solved problem, and not consistent enough to rely on. The habit of asking and understanding "could the model actually know this", needs to be owned by you, not relied on the model to be self aware.

---

At the end of the day, LLMs are a tool. The users that will get the most out of them are the ones who understand their capabilities and limitations and design their workflows and applications accordingly.

For applications where accuracy matters, you cannot rely on the training corpus or web retrieval alone. You need to supply the model with your own data and information through RAG, knowledge graphs, fine tuning, MCP tooling etc, so that the model has access to the information it needs to service your query.

I like to internalise this as the _"Dune Test"_. Before you trust an LLM to answer anything that matters, ask yourself this: "is the model likely to have access to this information?". Is it on the open internet, well cited and widely referenced? Is it locked in a book or a private data estate?

This will hopefully help you when asking LLMs questions and understand the importance of data accessibility when designing LLM based solutions.
