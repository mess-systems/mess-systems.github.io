---
title: "The legend, chapter 1. A company that does not exist"
date: 2026-10-22
weight: 3
description: "The first chapter of the mess.systems legend: a fictional company, a former B2B integrator with its own SaaS, starts rebuilding in the middle of the AI revolution."
---

> **This is fiction.** The company mess.systems described below does not exist. Its history, people and numbers are made up. It is a frame for the story: a fictional company is a convenient way to show how to rebuild work, people and systems in the move to the AI era. No real data from any employer goes into it.

The mess.systems [home page](/) has the word *mess* on it. Russell Ackoff used it for a situation made of interconnected, changing problems that cannot be taken apart and solved one by one. Describing such a situation in general terms is easy. Showing it on an example is harder: real companies do not talk about their arguments, and made-up examples in articles are usually too smooth.

So a company was invented. It has the same name as this project, and the things this blog writes about will happen to it.

## Why a fictional company

The company exists for idealized design. When you take part in different projects and try to see a company as a whole, having an idealized version of a company at hand helps a lot. With it, solutions can be proposed on the spot, and agents find it easier to work from a ready skeleton or template and adapt it to a real company. That way the context does not fall apart.

This is close to Ackoff's idealized design (the books "Redesigning the Future" and "Idealized Design"): you first design the system as you would build it today if the existing one were suddenly gone, and only then work from that picture back to reality.

## Ten years of billable hours

For about ten years mess.systems worked as a B2B IT integrator. It implemented accounting systems, electronic document management and service desks for clients, and built integrations between their systems. Revenue came from project hours.

On that experience the company built its own subscription SaaS. Revenue now had two parts: hours and subscriptions. The model was simple. More projects meant more people, and more people meant more hours.

## Then AI arrived

At first it looked like one more tool. Then clients began to notice that part of the work they paid for by the hour was now done by agents. An integration that used to take a week is put together in a day. Clients stop paying for hours that are no longer needed.

The SaaS has a similar story. Its functions are more and more often performed by an agent on top of any system: the agent does not need a separate interface, it needs a good API.

Both parts of revenue start to melt at the same time. The company has to rebuild, and it has to do it in the middle of the revolution, not after it.

## Two rebuilds at once

mess.systems works in two directions.

Inside, the company redesigns itself: how access, data, devices, infrastructure and people's roles are organized, and how agents fit into the work.

Outside, it turns its experience into a product. The product is called X, and it is simple on purpose. A user signs in, creates a record, sees a list of their records and deletes a record. A record has four fields: `title`, `text`, `date`, `owner`. There is a web interface, a public API and webhooks, subscriptions for individuals and for organizations, and an admin panel.

Why so simple. A complex domain is a distraction: the reader starts to study the product logic instead of how the product is built. A simple CRUD application shows what matters: how sign-in, data, payments, deployment and work with agents are organized. And questions without ready answers appear right away. What changes if a record is created not by a person but by an agent acting for them? Does the product need its own AI, or is a good API for other people's agents enough? How do you price a plan if the main API customer is an agent?

## Who is at the table

The company has nine executives. Each has their own responsibility, their own fear and their own interest.

- **CEO** is responsible for strategy and the company's survival. Fears the old revenue will melt before the company rebuilds. Wants to understand what clients will pay for once hours lose their value.
- **CFO** counts the money: tokens, hardware, people. Fears AI costs are growing faster than new revenue. Wants a clear price and a clear choice between own hardware and an external API.
- **CIO** is responsible for infrastructure, devices and the software catalog. Fears a zoo of devices and AI clients that nobody controls. Wants one catalog that shows what is installed, for whom and why.
- **CISO** is responsible for access and data. Fears an employee will send client data to an external chatbot. Wants to offer a safe, approved path to AI instead of bans.
- **CPO** is responsible for the old SaaS and product X. Fears the old product will die before the new one starts making money. Wants a product that is convenient for both people and agents.
- **CTO**, also the chief architect, is responsible for the target picture and technical standards. Fears decisions made by fashion. Wants a picture against which any decision can be checked.
- **COO** runs delivery, the project work. Fears the project model is melting while people and processes stay the same. Wants delivery in which agents do part of the work under human control.
- **CHRO** is responsible for people and their roles. Fears people will leave, or will be afraid of AI instead of working with it. Wants new roles and a clear path for everyone.
- **CDO** is responsible for data. Fears the data is scattered, has no owners, and agents are learning from garbage. Wants every data set to have an owner and rules.

## Why this is a mess

You could make a list: expensive tokens, uncontrolled devices, leaks to external AI, melting revenue, frightened people, data without owners. Assign each item to one of the executives above and give it a target.

But these problems are connected. If the CISO bans external AI, people will use it in secret, and the CIO gets an even bigger zoo. If the CFO cuts AI spending, the COO cannot move delivery to agents, and hours keep melting with nothing to replace them. If the CPO builds a product for agents, the CDO has to answer for data that is no longer created by people. Every executive can hit their targets while the company as a whole gets worse.

That is a mess in Ackoff's sense. It is not solved item by item. The company has to be redesigned as a whole, together with the people who work in it, so that some of the problems stop arising in their old form.

## What comes next

The legend will move in stages. First the target picture: what the company would look like if it were built from scratch today. Then the first decisions: access, names, devices, the path to AI, data, payments. Each chapter will have an argument between executives and the decision that came out of it. Then the first release of product X. At the end, an honest review: where the picture diverged from what actually happened.

The artifacts already published will become chapters of this story. The [naming standard](https://github.com/mess-systems/naming-standard) and the [tech radar](https://github.com/mess-systems/tech-radar) are the CTO's decisions. The [reference architectures](https://github.com/mess-systems/blueprints) and the [requirements template](https://github.com/mess-systems/requirements) are what product X will be built on.

What is not known yet: whether a fictional company can stay honest. Fictional executives find it easy to agree if the author wants them to. If an argument in the story seems too smooth, [write to us](/about/). You may recognize your own company in mess.systems, and then you will have something to add.
