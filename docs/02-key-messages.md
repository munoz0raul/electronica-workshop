# Key messages — the thesis and the four pillars

If I remember nothing else on stage, remember these. Every slide ladders up to one of them.

## The thesis

> **The edge can learn.** We take a device from *not knowing a word* to *knowing it
> and acting on it*, using data the audience generates, and we ship that new skill
> to a whole fleet over the air — the complete loop from data to deployed, live, in
> one hour.

The emotional arc is a question answered on stage: *slide 7 the device fails to
understand "blue"; slide 20 it understands "blue" and "red" — because the room
taught it and I shipped it.* Everything in between explains how that's possible
and why it's production-real, not a trick.

## The four pillars

### 1. One platform, prototype to production
The same app and the same model run on an accessible prototyping board (**UNO Q**)
and a production-class board (**VENTUNO Q**). That's the Dragonwing promise made
concrete: you don't rewrite when you graduate from bench to product.
*Lands on slides 3, 6, 13, 23.*

### 2. The edge learns from real data
We don't *program* the word "blue" — there is no `if` for the sound of a word. We
*show* the device examples, generated live by the audience, and it learns. Machine
learning is examples-in / model-out, and the quality of the model is the quality
and diversity of the data.
*Lands on slides 8, 9, 10, 11.*

### 3. Portability is an architecture, not a wish
The model and the app are identical across boards; only a thin hardware boundary
(mic device, MCU serial port) changes. Two containers separate the *app* from the
*model delivery*, so retraining is a file swap, not an OS rebuild. Yocto gives a
reproducible OS underneath.
*Lands on slides 12, 13, 14, 15.*

### 4. A maintained, secure update path is what makes it a product
Shipping to a fleet safely needs signed, versioned updates (TUF), a coordinated
rollout, and a fleet view to verify. App updates and OS updates run on separate
tracks — retrain all day without touching the base system, yet still ship a kernel
fix. This is the difference between a demo and something you can own for years.
*Lands on slides 16–21.*

## Lines worth landing (say them close to verbatim)

- *"You can't write an `if` statement for the sound of the word blue."*
- *"You are, right now, the training set."*
- *"Retraining is a file swap, not an OS rebuild."*
- *"The same artifact runs on a prototyping board and a production board — that's not a slogan, it's a file I can point at."*
- *"Twenty minutes ago it had never heard these words. You taught it, and we shipped it over the air."*
- *"The boring part — signed, versioned, won't-roll-backwards updates — is exactly what separates a demo from a product."*

## What I am NOT claiming (stay honest, pre-empt the sceptic)

- **Not** that the NPU runs this model. The voice model runs on the **CPU**; the
  40-TOPS NPU is the platform's headroom for bigger models, not what's doing the
  keyword spotting today. (See Q&A — this is the most likely "gotcha" question.)
- **Not** that this is a from-scratch model. It's **transfer learning** on a
  pre-trained keyword-spotting net — that's *why* it trains in minutes.
- **Not** that the live-trained model is guaranteed perfect. In a loud hall the
  honest move is a verified fallback model staged and ready (runbook). The *loop*
  is the message, not the recognition accuracy of one noisy session.
- **Not** that AI built this unattended. An agent did a lot of the typing under my
  direction; the architecture and judgment were human.
