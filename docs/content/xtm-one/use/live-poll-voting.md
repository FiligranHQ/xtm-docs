---
title: Let your audience vote from their phones
description: What the people in the room do and see when they answer the live poll of an XTM One deck from their phones, without an account, and what each message on their phone means.
doc_type: how-to
products: [xtm-one]
personas: [executive, cti-analyst]
owner: docs-xtm-one
last_reviewed: 2026-11-10
edition: ee
---

When a deck of XTM One holds a live poll, the people in the room answer from their own phones. They scan the QR code or type the short link of the slide, and a page made for a phone shows the question. They need no account and no app, and their answers are anonymous. The results on the slide follow within two seconds. This page tells you what your audience does and sees, so you can guide them and answer when a phone shows a message. How to add the poll to a deck and drive it is in [Ask the room with a live poll](/xtm-one/use/presentations/#ask-the-room-with-a-live-poll).

::screenshot{id=xtm-one-poll-vote alt="The voting page of the crisis cell's poll as a phone opens it: Live poll at the top, the question Which decision comes first?, the three decisions as options, the Vote button, and under it the count of each option after the first three votes."}

## Before you start

- **A deck with a live poll**: see [Ask the room with a live poll](/xtm-one/use/presentations/#ask-the-room-with-a-live-poll). Presentations need the Enterprise Edition.
- **Public presentation links stay on**, which is the default: the voting page is public. When an administrator turns **Public presentation links** off, every phone reads "This poll is not available". See [Show it to people without an account](/xtm-one/use/presentations/#show-it-to-people-without-an-account).
- **Phones reach XTM One**: the short link is your platform's address, then `/v/` and the poll's code, such as `https://xtm-one.grand-meridian.example/v/HX4TQ`. When XTM One answers only on the company network, a phone on mobile data or on the guest Wi-Fi cannot open it.

## Get the room to vote

:::steps
1. Present the slide that holds the poll. It shows the short link and, when **Show the QR code** is on, its QR code.
2. Ask the room to scan the QR code with their phone's camera, or to type the link. The code has five letters and digits, never 0, O, 1, I or L, so nobody confuses them when you read it aloud.
3. Each person answers on the page that opens, under **Live poll**: see [Answer each kind of question](#answer-each-kind-of-question).
4. Watch the results on the slide. When the room has answered, click **Lock voting**, then read the result aloud.
:::

## Answer each kind of question

| Question | On the phone |
| --- | --- |
| **Choice** | Tap an option, then **Vote** |
| **Quiz** | Tap the answer you think is right, then **Vote** |
| **Rating 1-5** | Tap a number from 1 to 5, then **Vote** |
| **Word cloud** | Type in the field, then **Send**. The field says **One word**, or how many words the question takes, separated by commas |

Once the vote is in, the buttons give way to "Thanks, your vote is in. Look at the screen." A word cloud keeps each word in lower case, cut at 30 characters, and a word typed twice counts once. A rude word refuses the whole answer with "That word is not accepted.": the person types another one.

How the questions reach the phones depends on the poll's **Mode**:

- **Guided**: the phone shows only the question you opened, and moves on when you do. Each new question comes with its own buttons, also for the people who answered the previous one.
- **Autonomous**: the phone lists every question on one page, each with its own **Vote** or **Send**. People answer in any order, at their own pace, and you drive nothing.

## One vote per phone, and nobody named

- Each phone answers each question once. The page keeps a random token in the phone's browser, and a second vote from it is ignored. XTM One records the vote with a scrambled form of that token, never a name or an account.
- This stops a double tap, not a person who changes browser: another browser counts as another phone. Read the result as the opinion of the room, not as a formal ballot.
- A poll takes 500 phones at most, over all its questions. The next phone reads "This poll has reached its number of voters."; a phone that voted already can still answer the other questions.
- The phone remembers its answers in its browser. Rehearse from a private window: a phone that voted during the rehearsal still shows its thanks line after **Reset votes**.

## What the phone shows after the vote

- **Results** on **Real time**: under the question, the number of votes for each option or each rating, refreshed every two seconds. On **After lock**, the numbers show once you lock voting. The words of a word cloud show on the slide only.
- **Lock voting**: the phones read "Voting is closed for this question.", with the numbers. **Reopen voting** gives the buttons back to the phones that have not voted.
- **Quiz**: the right answer stands out among the numbers once it is revealed, with **Reveal the answer** or as the poll's **Quiz answer** sets it.

## When a phone shows a message

| The phone reads | Why | What to do |
| --- | --- | --- |
| "This poll is not available" | The code is mistyped, the deck or its poll was deleted, public presentation links are off, or too many phones read the poll through one network | Check the link on the slide. Ask an administrator whether public presentation links are on |
| "Waiting for the presenter to open a question." | No question of the poll is open | Open one with **Next question** or **Previous question** |
| "This question is not open." | The person answered a question you had just moved past | They answer the question their phone shows now |
| "Voting is closed for this question." | You locked voting | **Reopen voting** when you want more answers |
| "This poll has reached its number of voters." | 500 phones answered already | Ask the next questions in a second poll, which takes 500 more |
| "Your vote could not be sent. Try again." | The phone lost the network, or too many votes came through one network at once | Tap **Vote** or **Send** again |

Every phone reads the poll every two seconds. The phones that reach XTM One through one network address, such as one guest Wi-Fi, may read it 6,000 times and vote 1,200 times a minute: about 200 phones. In a larger room, ask people to use their mobile data.

:::example[Let the crisis cell pick the first decision]
On Monday 13 April 2026, Hélène Marchetti, CISO of Grand Meridian Hotels, runs the crisis cell's meeting on Velvet Harbor. Her deck *Crisis cell vote on the three decisions* holds one **Choice** question, "Which decision comes first?", in **Guided** mode, with **Results** on **Real time**.

She shows the slide and says: "Scan the code, or type xtm-one.grand-meridian.example/v/HX4TQ." The managers of the Paris, Madrid and Barcelona hotels open the page on their phones, without signing in. Each taps a decision, then **Vote**, and reads "Thanks, your vote is in. Look at the screen."

On the slide, **Notify the guests concerned** gets two votes and **Rehearse the scenario** one. Hélène clicks **Lock voting**: the phones read "Voting is closed for this question." with the three counts, and the cell starts with the notification of the guests.
:::

## What's next

- [Turn an agent's work into a presentation](/xtm-one/use/presentations/): build the deck, add the poll and drive it from the speaker view.
- [Measure what your agents' work is worth](/xtm-one/use/value/): a deck whose poll you drove counts as used on the **From chat, estimated** card.
