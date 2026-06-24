# TakeMeter — Planning

## Community
I chose **r/LetsTalkMusic**, a subreddit built specifically for in-depth music
discussion rather than sharing songs or news. It's a strong fit for a
classification task because its discourse varies widely in quality: the same
thread can contain a deeply argued essay, a one-line hot opinion, and a
recommendation request. That natural spread of "how substantive is this post"
is exactly the distinction the classifier needs to learn.

## Labels
I use three mutually exclusive labels. The core test is **what the post is mainly
doing**: arguing, asserting, or asking.

**argument** — The post makes a structured case for a claim, backed by specific
evidence: musical detail (production, lyrics, structure), historical/genre
context, or comparison to other works. The reasoning is visible even if you
disagree.
- Example: "Kid A worked because they replaced guitar structures with modular
  synth textures, echoing what Aphex Twin did years earlier."
- Example: "Mainstream country flattened in the 2010s as Nashville consolidated
  writing around a few teams, making the instrumentation formulaic."

**opinion** — A clear stance or preference stated without real supporting
evidence. The claim may be correct, but the post asserts rather than argues.
Includes recommendations and "overrated/underrated" claims with no backing.
- Example: "Tame Impala peaked with Lonerism and everything after is a downgrade."
- Example: "If you like Phoebe Bridgers, check out Julien Baker — similar vibe."

**question** — The post primarily asks something: a recommendation request, a
request for explanation, or a discussion prompt. The poster wants input, not to
deliver a take.
- Example: "What albums are perfect front-to-back with zero skips?"
- Example: "Why is shoegaze having a revival right now?"

## Hard Edge Cases
**A question that's really an opinion in disguise:**
"Why does everyone pretend Sgt. Pepper is a masterpiece when it's clearly worse
than Revolver?"

This opens like a question but is pushing a stance.

**Decision rule:** If removing the question mark leaves a clear assertion the
poster obviously believes, it's `opinion`. A genuine `question` is one where the
poster would accept "you're wrong" as a valid answer. → This case = `opinion`.

Other ambiguous cases encountered during annotation:
- **Short posts citing one stat** — sits between argument and opinion. Rule: if
  the evidence genuinely supports the claim, label argument; if the stat is
  decorative or cherry-picked, label opinion.
- **Recommendation requests that include a strong opinion** ("I love X, what
  else is like it?") — labeled question, since the main act is asking.

## Data Collection Plan
I collected 200 public posts/comments from r/LetsTalkMusic by reading threads and
copy-pasting text into a spreadsheet with columns text/label/notes. I targeted a
spread across all three labels. If a label came out underrepresented, the plan was
to open recommendation and discussion threads to collect more of that class until
no label exceeded 70%.

## Evaluation Metrics
- **Overall accuracy** for a quick headline number on both models.
- **Per-class precision, recall, and F1** — accuracy alone hides whether one class
  is ignored. Since the argument/opinion boundary is the hard one, per-class F1
  shows whether the model learned that distinction or leaned on an easier class.
- **Confusion matrix** to see the direction of errors (is it calling arguments
  opinions, or the reverse?).
F1 per class is the metric I trust most because the task is subjective and the
classes are not perfectly balanced.

## Definition of Success
For a real community tool, I'd want overall accuracy meaningfully above the
zero-shot baseline, with every class at F1 ≥ 0.65 and the hardest pair
(argument vs opinion) at F1 ≥ 0.60. Below that, the tool would mislabel takes too
often to be useful. I'd accept ~0.70 macro-F1 as "good enough" for a first
deployment, with known failure modes documented.

## AI Tool Plan
- **Label stress-testing:** I gave an AI my label definitions and edge-case rule
  and asked it to generate boundary posts between argument and opinion. Where it
  produced posts I couldn't classify cleanly, I tightened the definitions before
  annotating.
- **Annotation assistance:** I labeled examples myself by reading each one.
- **Failure analysis:** After fine-tuning, I used an AI to help interpret the
  confusion matrix and identify the error pattern, then verified it against the raw
  numbers myself.
