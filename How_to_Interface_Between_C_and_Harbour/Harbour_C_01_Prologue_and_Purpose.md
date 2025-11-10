---
book_title: "Buccaneers 2 - Harbour and the C Language"
chapter: 1
chapter_title: "Prologue and Purpose of the Course"
source: "Bucaneros2.odt"
author: "Manu Expósito Suárez"
language: "English (translated from Spanish)"
translator: "ChatGPT (GPT-5)"
notes: "Faithful translation preserving technical accuracy and structure. Neutral tone."
---

# 1. Prologue and purpose of the course

## Foreword by Rafa Carmona

One day, chatting with Manu Expósito, he mentioned the title of a book I wrote more than 19 years ago: *The New Buccaneers of the Planet, Part 1*.
It was a book conceived and created simply for myself, opening with an introduction—among other things—about the magic that existed in the communication between the C language and Harbour, and it was released free for anyone interested in the subject.

Why do I bring this up? Because he told me he wanted to use the same name.
“Well,” I told him, “my book was Part 1, and if you want to use the same or a similar name, then you should call it Part 2, because Part 2 has been pending for more than 19 years—and it would be an honour for me if this became that Part 2.”

I have known Manu Expósito for many years, and I know his stubborn determination to push Harbour’s performance to the limit—and for that, C is king.

Much has happened since then, and with it the system between C and Harbour has been expanded and improved; Manu Expósito’s Part 2 is needed now more than ever.

I have seen the syllabus that Manu is preparing for this Part 2 of *Los nuevos bucaneros*, and I know it will not disappoint—no one will be left indifferent.

Manu Expósito is one of the people who knows the most about the connection between C and Harbour. He wrote his famous **HDO** library in pure C—yes, everything is built on the C language and connected with Harbour. Even the classes you use in HDO have been created from C. You can be sure that what you are holding in your hands is pure gold, and it will take away that fear we all feel when we enter an unknown and almost magical world like the C Language, the undisputed king. Long live C! Long live Harbour!


## And now, my part… (by Manu Expósito)

For many years I’ve wanted to write some notes to use personally.

A long time ago, I mentioned this to a colleague, and he suggested that those notes could become a book and be published on some platform so that anyone interested could pay a small amount and benefit from it.
That would also be a reward for me.

I tried it at the time, but I don’t think I was very enthusiastic.
Now the necessary circumstances are coming together to roll up my sleeves and start this adventure.
I simply feel like doing it…!!!

Back in the day, my friend Rafa Carmona wrote a book called *Bucaneros*. That book covered everything related to programming in Harbour but at a high level in PRG, with a small reference to Harbour’s extended system. By the way, I’ve asked Rafa if he wants to write a few notes for us, which I’ll gladly add to this prologue.

A lot has happened since then… Harbour has come of age, and the extended system is now very mature. It’s time to discuss these topics in depth.

I must say that Harbour includes “out of the box” almost everything we may need—and in a very optimal way. So it will be **almost** unnecessary for us to enter the seas of C.
But this book is written for that *almost*.

For example, if we want to integrate dynamic link libraries (DLLs), or develop routines that make certain critical bottleneck functions faster, it will be necessary to reach for the omnipresent C language.
After all, Harbour itself is built in C.

Luckily, Harbour inherits from the legendary Clipper the **Extended System** and the **ITEM API System**, which is the method by which we can do “magic.” And unlike what happened with Clipper, in Harbour there are no undocumented black boxes—we have all its source code available to us.

The purpose of this book is to serve as a **logbook** for this voyage—a journey we know how it begins, but not how it ends, because it is yet to be written. This book is alive; it will be fed by what you all contribute.

The idea is that I will write it topic by topic and deliver it to you in **PDF** and **ePub** format so you can use it as a notebook, expanding what is necessary. If you want, you can later send your additions back to me to enrich what will eventually become the book—once finished, all registrants will receive it.

The ultimate goal is that when the voyage across the **Seven Seas** is over, we will all be capable of writing our own functions in C to improve the robustness and performance of our programs, and we’ll be able to create our own libraries…

**So… let’s not waste a single second!!!**

The curtain opens and the adventure begins…
