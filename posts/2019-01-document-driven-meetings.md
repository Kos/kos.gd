<!--
.. title: Document-Driven Meetings
.. slug: document-driven-meetings
.. date: 2019-01-18 12:35:00 UTC
.. tags: meetings
.. category: dev
.. link:
.. description:
.. type: text
-->

In my [previous post][prev], I argued that meetings tend to much better if they have a driver and a well-defined objective.

I'd like to invite you to a little thought experiment. Assume there's only one possible meeting objective: to collaboratively write a document. Can you fit each of your meetings into this shape?

<!--more-->

## Applying the metaphor

Many meetings, even when they don't look like like it first, are in essence all about creating something. That 'something' can often be given a form, and prose is a convenient option.

Here's some meetings I typically have at work and how they fit into this metaphor:

- Daily stand-up meeting is where we write a plan for the day.
- Iteration planning is where we write down a weekly goal and planned way to get there.
- Refinements and design workshops are where we write down a spec for something we're planning to build.
- Team retrospective is where we write down the list of team commitments: what we're going to change to improve how we work.
- A job interview is a meeting where as an interviewer I write down my observations about a candidate, or the other way round. (In practice, I'd scribble during the interview and write the proper doc afterwards.) This case is a bit funny: two objectives imply two independently written documents.

That's actually plenty!

I only found one general case where the metaphor clearly doesn't apply: Meetings where you learn things or teach things, such as conference talks or onboarding sessions. In this case the objective is simple: the audience should walk out of the meeting smarter than they came in. This objective is not about creating anything. Of course you can still take notes, but the notes are not a document in this context, just the extension of your own memory.

Another tricky case is [1-1 meetings][11meeting], a general idea which has wildly various formats in practice. Typical themes are problem-solving and guided personal development. Both fit into the "planning meeting" theme, where the goal is to produce a list of action points for either participant. However, this is a very inhumane way to put it, since a vital element of a good 1-1 meeting is bonding and learning to trust each other. This shows that even if no document is produced, a meeting can still be valuable in its own way.

---

Now the imporant point:

> Magical things happen if you go through the trouble of actually writing the document during the meeting (or at least, directly afterwards).

Some reasons why:

## Persistence

The most obvious benefit of putting the outcome down in writing: Once the document's written, it stays there. You can come back to it at any time, search it and review the meeting outcome.

If a decision is made, you obtain a record of who took this decision and when. On top of that, you'd typically write down what the reasoning was and what other options has been considered (possibly in a form similar to ["rejected alternatives"][pep] from Python Extension Proposals. This kind of insight is usually easier to lose unless written down, even when the decision itself stays.

But most importantly, you have a record of what decision, exactly, has been made. This leads to the next point.

## Clarity

> "Does everyone agree with this picture? Yes? Great, let's move on."

If a decision needs to be taken by a group, then unless you write it down, there's no good way to guarantee that everyone has the same thing in mind.

You can draw a diagram on the whiteboard, have a disagreement about various elements, make corrections, and then do a quick round of voting (or a [decider][decider]): does everyone agree? Then perhaps everyone says yes, then - fast forward two weeks - you refer to the discussion, and someone replies "but I thought we agreed that this means something else".  Disagreement ensues and the discussion goes nowhere. On the surface you had agreement, but underneath, the decision was still not clear.

Now write the same thing down as a plain sentence and ask: "Does everyone agree?" Now chances are the dynamics will be much different: Someone might say "I agree with most of it except that small finer point", and then you might split the sentence into two, agree on the first half and discuss the rest separately. Or someone might say "Wait, what does Some Term mean exactly in this context? I don't think I can fully say if I agree until we clarify that". You'd clarify further and refine the formulation.

A sentence, once written and displayed, leaves less space for misinterpretation than anything said out loud or demonstrated with a diagram.

## Purpose

A document easily ensures that the meeting's objective is clear to all participants. It removes a lot of gray zone about what is in the scope of the meeting and what isn't. It also sets a clear finish line that shows when the meeting is over.

The same thing can be accomplished by writing down and sharing a specific agenda before the meeting happens (which is a good idea!). But if you take this agenda and convert each bullet point into a header, you already have a good document draft that you can collaboratively flesh out during the meeting.

The document is in flux for the whole duration of the meeting. If you observe that your expectations for the meeting don't match what's in the document outline, there's still time to suggest to extend (or narrow down) the meeting scope.

## Practical concerns

Document-driven meetings only work if everyone's looking at the same document. There are two setups that can accomplish this:

1. Everyone has the document open on their own laptop.
2. There's a big screen visible to everyone & someone is presenting.

Both setups require you to use some software that allows to collaboratively edit text (I use Dropbox Paper or Google Docs). Even if you have a single presenter, all participants should still be able to connect and edit. (Having to dictate destroys meeting flow.)

A dedicated presenter is not required but often useful, because the large screen makes it clear which part of the document is being worked on at any given moment.

Note-taking during the meeting is often hard. If you have this problem, you can mitigate it by assigning a specific person for writing down what's being discussed. Another solution is to write the document just after the meeting itself & share it with participants and ask for review & extension.

(The Document-driven meeting pattern seems to have potential to make meetings easier for remote participants, but I don't have enough experience with this format yet to reliably confirm this.)

## Wrap up

Document driven meetings fix the problem of unclear meeting objective by giving the objective a physical, tangible form. This reduces risk of objective mismatch (or more generally, expectation misalignment) between meeting participants.

The document-driven meeting is a metaphor. It's a handy concept to use even if you don't end up physically writing anything down. (Why not though...?)

As a developer, I like document-driven meetings because they feel more "like work" to me. This might be because it's more similar to what I normally do while programming: think and write stuff. Using prose instead of code is just a detail.

Finally, converting a meeting to a document-driven meeting is usually easy. First step is to bring a laptop and volunteer to take notes.

[11meeting]: https://a16z.com/2012/08/30/one-on-one/
[decider]: https://liveingreatness.com/core-protocols/decider/
[pep]: https://www.python.org/dev/peps/pep-0484/#rejected-alternatives
[prev]: /posts/efficient-meetings/
