# Preface

## Software Wrote This

A language model wrote this book. Every chapter, every table, every sentence.

That belongs at the front rather than in a note at the back, because it changes what you should do with what follows. **Check things.** Every claim taken from Anthropic's documentation carries a numbered reference to the page it came from, and those references exist so you can open them, not so a paragraph looks well dressed. A book written by software that asks to be believed is asking for something it has not earned.

What software turns out to be good at is the job this book needed done: reading a large body of documentation closely, holding all of it at once, and noticing where two pages say different things. There are places in here where they do. Where that happened, this book says so and names which page it followed, rather than picking the tidier one and moving on.

What software is bad at is knowing the edge of what it knows. That is what the references are for, and it is why one instruction outranked every other rule this book was written under: **no invented specifics**. No company, no customer, no incident, no benchmark, no measurement that is not in a source you can go and read. Where a scenario appears it is openly hypothetical, and where a figure appears it came from somewhere named.

## Who This Is For

You have the exam booked, or you are deciding whether to book it. That is the only reader considered here, and it settled every argument about what to include.

The exam is **53 items in 120 minutes**, multiple choice and multiple response, each item telling you how many answers to select. It is scored 100 to 1,000 and **720 passes**. Behind it sits a published blueprint of **8 domains and 25 skills**, with a weight given for each skill rather than only for each domain, which is finer detail than most certifications hand out. This book is organised around that blueprint and the back matter maps every skill to the chapter that owns it.

One rule governed the writing. Every paragraph had to **earn its place against something the exam tests**. Not against being interesting, not against being complete, not against sounding thorough. A passage that would not help you answer something you would otherwise miss is not in here.

There is good writing on building with these models and almost none of it is arranged around what you will be asked. This is not a survey. It is roughly **72,000 words** aimed at 25 named skills, which is a weekend to read and an evening to skim again. The length is a decision, not an accident: **a book you finish beats a book you admire**, and the ground is wide enough that a complete treatment would never be read by the person who most needs it.

## What the Exam Is Actually Like

Two properties of this exam shape the whole book, and both are worth knowing before chapter 1.

**Nearly every wrong answer is a mechanism that works.** Options are not built from nonsense. They are built from real features used one step out of place: a Skill where a server was needed, a context file where a hook was needed, a retry where the failure was permanent. So knowing that a mechanism exists is worth very little here, and knowing what it is for is worth almost everything.

**A third of it is not about Claude.** Domain 2 alone is **33.1%** and it covers REST semantics, versioning, review discipline and application boundaries. Domain 4, which covers evaluation, testing and debugging, is **2.6%**. That is a thirteen-fold spread, and a reader who budgets study time by how interesting a topic is will spend their evenings in the wrong place. The coverage map at the back prints every weight so you can spend them deliberately.

## How a Chapter Is Built

Every chapter has the same parts in the same order, so you can drop into one you have not read.

An **italic summary**. What decision the chapter settles, and what getting it wrong costs.

**Sections whose headers state their point.** No teasers. A header that means nothing until you have read the section under it is dead weight in a book somebody revises from, and if you find one it is a defect.

**Tables wherever the material is parallel.** A table does in forty words what prose needs four hundred for, and there is a lot of ground here.

**Named failure shapes, in bold.** These carry more than they look. Wrong answers are built out of plausible mistakes, and a mistake you can name is one you can spot in an option list you have never seen.

A section called **How Wrong Answers Are Built on This Material**. Families of assumption that wrong answers rest on, then a table: what the scenario says, what the answer usually is, what the trap usually is. Those rows are written as **cues rather than cases**, so they still fire on a scenario set in an industry this book never mentions.

A closing list called **What to Remember**. Marked densely, and written so every line stands up with the chapter shut, because that is the one condition under which anybody reads such a list.

## How to Read It

**With time**, read in order. The chapters follow the sequence you meet the decisions in when you build something, so each leans on the last.

**Without time**, read the closing lists first, all twenty-three, then read the chapters behind any line you could not defend on your own. The book was built to survive being used that way.

**Chasing one weakness**, go to the coverage map, find the skill, and read the chapter that owns it.

One honest caveat. **Anthropic's documentation moves.** Numbers change, features ship, pages get reorganised. Everything cited here was read on the dates in the references and some of it will have changed by the time you read this. The reasoning outlives the figures, which is why the reasoning is where the pages went. When a number matters to your answer, follow the link.

Now go and be wrong cheaply, here, rather than expensively, in an exam room.
