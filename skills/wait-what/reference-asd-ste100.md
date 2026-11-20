# ASD-STE100 checklist for learning skill output

Every message a learning skill sends the developer follows these rules.
This covers explanations, multiple-choice questions, option labels and
descriptions, hints, and recap lines.

Where a skill defines a level voice (for example `explain`'s
`five` or `teammate`), that voice sets vocabulary depth, analogy use,
and whether to show code. These sentence and word rules still apply on
top of it.

## Sentences

- Keep sentences short. Aim for 20 words or fewer.
- Put one idea in each sentence.
- Use active voice. Write "The loop reads the list." Not "The list is
  read by the loop."
- Write instructions as commands. Write "Check the null case first."
  Not "The null case should be checked first."
- Start a new sentence for a new idea. Do not join ideas with "and" or
  a semicolon.

## Words

- Use plain, common words. Use "use" not "utilise". Use "start" not
  "initialise". Use "about" not "regarding".
- Use one word for one meaning. Pick a term and keep it. Do not call
  the same thing a "list" in one line and an "array" in the next,
  unless the language forces the difference.
- Use one meaning for one word. Do not use "close" for both "shut" and
  "near".
- Cut words that add nothing. Cut "actually", "simply", "just", "in
  order to" (use "to"), "at this point in time" (use "now").
- Do not use idioms. Do not write "get the ball rolling" or "corner
  case". Write "start" and "edge case".
- Explain a technical term the first time you use it, in one short
  sentence.

## Structure

- Prefer a short list to a long paragraph.
- Keep each list item to one instruction or one fact.
- Keep a paragraph to three sentences or fewer.

## Tone

- Be direct and calm. No praise words like "great" or "perfect".
- Do not use "!" for emphasis.
- Ask the question. Do not add a preamble like "Good question."

## Asking questions

When a skill says "ask one question", it means one multiple-choice
question to the developer.

- **Use the CLI's own question tool.** Most agent CLIs have a tool for
  asking a multiple-choice question, under different names. If one is
  available, use it. Fill in what it supports: a short title, a
  description for each option, a preview. Leave out what it does not
  support.
- **If there is no such tool,** print the question and a numbered list
  of options. Then end your turn and wait. Accept the number, the
  label, or free text. Never move on without an answer.
- **Never mark the correct option.** Do not bold it, order it by
  correctness, or hint at it in the label.
- **Free text is always allowed.** Treat a typed reply, or a pick of an
  "Other" option the tool adds, as the developer's own answer.
- **Ask one question per turn** unless the skill says to ask several at
  once.

## Quick self-check before sending

1. Is any sentence longer than 20 words? Split it.
2. Is any sentence passive? Make it active.
3. Is there an idiom or a rare word? Replace it.
4. Is the same idea said twice? Cut one.
5. Does an option or choice describe one clear wrong idea, or is it
   vague filler? Make it specific.
