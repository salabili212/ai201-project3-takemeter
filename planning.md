# TakeMeter — Classifying Discourse Quality in r/LetsTalkMusic

## Community
r/LetsTalkMusic is a subreddit for in-depth music discussion. I chose it because its posts
range from deeply argued essays to one-line opinions to recommendation requests — a natural
spread of discourse quality that makes classification meaningful.

## Label Taxonomy
- **argument** — a structured case backed by specific evidence (musical detail, history, comparison).
  - Ex: "Kid A replaced guitar structures with synth textures, echoing Aphex Twin."
  - Ex: "Country flattened in the 2010s as Nashville consolidated songwriting teams."
- **opinion** — a stance stated without real evidence; asserts rather than argues.
  - Ex: "Tame Impala peaked with Lonerism."
  - Ex: "Like Phoebe Bridgers? Try Julien Baker."
- **question** — primarily asks for recommendations, explanation, or discussion.
  - Ex: "What albums are perfect front-to-back?"
  - Ex: "Why is shoegaze reviving right now?"

## Dataset
- **Source:** 200 posts and comments collected manually from r/LetsTalkMusic.
- **Labeling process:** I read each post and applied the definitions above, recording ambiguous cases in a notes column.
- **Label distribution:** argument 101, opinion 62, question 37.
- **Balance note:** the dataset is imbalanced — argument is ~51% and question only ~18%.
  With the 70/15/15 split, the test set is 30 examples (15 argument, 10 opinion, 5 question),
  so per-class metrics — especially for question — rest on very few examples.

## Fine-Tuning Approach
- **Base model:** distilbert-base-uncased
- **Training setup:** 3 epochs, learning rate 2e-5, batch size 16 (defaults).
- **Hyperparameter decision:** I kept 3 epochs because with only 140 training examples, more
  epochs risk overfitting — the model would memorize training posts rather than learn the
  argument/opinion/question distinction.

## Baseline
- **Model:** Groq llama-3.3-70b-versatile, zero-shot.
- **Prompt:** A system prompt giving the three label definitions, one example each, the
  question-vs-opinion decision rule, and an instruction to respond with only the label name.
- Results collected by classifying every test example and comparing to true labels.

## Evaluation Report

### Overall Accuracy
| Model | Accuracy |
|---|---|
| Baseline (Groq zero-shot) | 56.7% |
| Fine-tuned DistilBERT | 36.7% |

The fine-tuned model performed **worse** than the baseline (−20 points). This is the most
important finding of the project and is analyzed below.

### Per-Class Metrics — Baseline
| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| argument | 0.62 | 0.33 | 0.43 | 15 |
| opinion | 0.41 | 0.70 | 0.52 | 10 |
| question | 1.00 | 1.00 | 1.00 | 5 |

### Confusion Matrix — Fine-tuned
| True \ Pred | argument | opinion | question |
|---|---|---|---|
| **argument** | 9 | 6 | 0 |
| **opinion** | 6 | 2 | 2 |
| **question** | 4 | 1 | 0 |

### What the confusion matrix shows
The fine-tuned model collapsed toward predicting **argument**: it predicted argument 19 of 30
times, opinion 9 times, and **question 0 times** — it never once predicted question, despite 5
question posts in the test set. It correctly caught 9/15 arguments but misread most opinions and
all questions as argument.

### Three Wrong Predictions Analyzed
1. **A question post → predicted argument.** The model predicted question zero times overall.
   With only ~37 question examples in training (and ~26 after the split), the model saw too few
   to ever learn the class, so it defaulted to the majority class.
2. **An opinion post → predicted argument.** Opinion and argument share music vocabulary (artist
   names, album titles). Without enough examples to learn that *evidence/structure* — not topic —
   separates them, the model leaned on the most frequent label.
3. **An argument post → predicted opinion.** Even within the class it favored, the argument/opinion
   boundary is subtle; short arguments citing one detail look like bare opinions, which the model
   couldn't reliably separate.

### Sample Classifications
| Post (abbreviated) | True | Predicted |
|---|---|---|
| "Kid A replaced guitar structures with synth textures..." | argument | argument |
| "Tame Impala peaked with Lonerism." | opinion | argument |
| "What albums are perfect front to back?" | question | argument |

The first prediction is reasonable: it's a clearly structured, evidence-backed claim, exactly
what the argument label is meant to capture — and the one class the model could identify.

## Reflection: Learned vs. Intended
I intended the model to separate posts by *how substantive the reasoning is*. What it actually
learned was far simpler: **predict the majority class (argument) most of the time.** It did not
learn the question class at all and only weakly distinguished opinion. The gap comes from two
things: (1) too little data — 140 training examples is very small for a subjective 3-way task —
and (2) class imbalance, which gave the model an easy shortcut (guess argument) that beat
actually learning the boundaries. The zero-shot LLM, with broad pretraining, handled the task
better precisely because it didn't depend on my small, skewed training set.

### What would fix it
More data, especially for question and opinion to balance the classes; and possibly class
weighting during training so the model is penalized for ignoring minority classes.

## Spec Reflection
The spec helped by making me write my edge-case decision rule before annotating, which kept my
labels consistent. My implementation diverged from the ideal in that my dataset ended up
imbalanced (argument-heavy), which the spec warned against — and that imbalance turned out to be
the main driver of my model's failure, exactly as the spec predicted.

## AI Usage
1. **Label design:** I used an AI to generate boundary posts between argument and opinion and
   tightened my definitions where I couldn't classify them cleanly.
2. **Failure analysis:** I used an AI to help interpret my confusion matrix and identify the
   majority-class-collapse pattern, then verified it against the raw numbers myself.

