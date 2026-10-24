---
title: "The screen as an attack vector"
date: 2026-10-24
weight: 4
description: "Predictions: screens are now watched by machines that understand what they see and act on it, and the screen is becoming an attack vector."
---

# The screen as an attack vector

It started with a simple idea for a program of my own. I wanted to take part of the screen, say the window where an agent is working or a terminal is running, and send the video from that part to another agent. A specialized one that does nothing but watch it and speak up if something goes wrong. Basically an automated version of what I already do by hand when I ask one agent to check another agent's output.

Then a question came: what if the same stream from my screen goes not to my agent, but to an attacker?

It seems to me the screen is becoming a very real attack vector. Not because a new vulnerability has appeared, but because screens are now watched by machines that understand what they see and act on it.

## Screen streams will be everywhere

Until now, the picture on a screen was mostly needed by the person in front of it. Now many will want it, and all of them will have good reasons.

Defenders will put a watcher agent next to the working agent to catch dangerous commands and leaks. Employers can already take screenshots of desktops, and soon a model will look at them, with conclusions attached. Governments have long had access to communications: in Russia operators are required to install SORM equipment, other countries have their own interception laws. If every computer runs an agent that sees the screen, this source is unlikely to be left alone.

Every such stream is one more copy of what you see. One more place it can be stolen from.

## What an attacker can do

**Steal the stream.** If someone gets the video of your screen, they need nothing else. Passwords as you type them, chats, code, documents, SMS codes all pass in front of the camera by themselves. A recording like that used to need someone to watch it. Now a model will watch it, fast and without getting tired, and pull out only what matters.

## Where these thoughts about hacking come from

There is a lot of fear around AI right now, and in my view the biggest players like OpenAI and Anthropic noticeably fuel it. They need huge amounts of money for data centers, and strict rules they help write conveniently cut off smaller competitors. This is my opinion, but some of it shows in the numbers: according to [POGO](https://www.pogo.org/investigates/boom-in-major-ai-company-spending-on-lobbyists), Anthropic spent 1.04 million dollars on federal lobbyists in the US in 2024 and 8.11 million in 2026 through the end of the third quarter, OpenAI 2.45 million and 4.77 million for the same periods. I think I stole or overheard this idea from Steve Eisman. In the episode [“Why Dario Amodei and Sam Altman Are Faking the AI Doomsday Crisis”](https://www.youtube.com/watch?v=Q1jSJG1iBL8) he says Anthropic and OpenAI are fomenting hysteria and inviting the federal government to regulate the industry, because they see a price war coming and hope regulation will protect their pricing and create an “AI duopoly”.

At the same time, vulnerabilities existed before, people found them before, and it always mattered. The noise around AI does not make the risk imaginary. It just makes it harder to tell the loud from the real. And a screen that agents look at is the real thing.

It reminds me of the story of leaded gasoline. In 1921 a General Motors team found tetraethyl lead: it really did save engines from knocking and made enormous money. Factory workers started dying in the very first years, but the industry defended the product for decades with its own research, pressure on scientists and lawsuits ([EPA](https://www.epa.gov/archive/epa/aboutepa/lead-poisoning-historical-perspective.html)). Clair Patterson, while measuring the age of the Earth, found lead everywhere and showed that the "normal background" was itself pollution ([Caltech](https://magazine.caltech.edu/post/clair-patterson-lead)). After his 1963 paper in Nature, he was threatened with losing his funding ([UChicago Magazine](https://mag.uchicago.edu/science-medicine/immeasurable)). In the US the ban for road vehicles took effect in 1996, and worldwide Algeria was the last to give it up, in 2021 ([UNEP](https://www.unep.org/ru/novosti-i-istorii/istoriya/po-sledam-20-letney-kampanii-izbavleniya-mira-ot-etilirovannogo-topliva)).

AI is not lead, of course, but it also needs to be adopted carefully. Watching what the rich companies do matters a lot. And so does coming together around these problems.

## Screens are already being recorded

This is not fantasy. Smart TVs use ACR, automatic content recognition: they take fingerprints of the picture and sound and compare them with a database to find out what is being watched at home. The Texas Attorney General, in a [lawsuit against LG](https://www.texasattorneygeneral.gov/sites/default/files/images/press/LG%20TV%20Petition%20Filed.pdf), writes that ACR stores a "hash" every 10 ms and works with all inputs, including HDMI, even when the TV is used as a computer monitor. A similar [lawsuit was filed against Samsung](https://www.texasattorneygeneral.gov/sites/default/files/images/press/Samsung%20TV%20Petition%20Filed.pdf). [Gamers Nexus](https://www.youtube.com/watch?v=6IFVTcM28KA) has a detailed breakdown of LG TVs. LG disagrees with the findings and [says](https://www.techradar.com/televisions/lg-tvs-collect-far-more-data-on-you-than-youd-expect-says-new-report-including-logging-microphone-audio-while-on-standby) that ACR is only turned on with the user's consent.
