# Breaking a Spam Filter

A simple SMS spam classifier built with **TF-IDF + Multinomial Naive Bayes** (scikit-learn).
I trained it, measured it against a baseline, then wrote messages designed to fool it
and explained why each one worked.

## Dataset

[SMS Spam Collection](https://archive.ics.uci.edu/dataset/228/sms+spam+collection) (UCI):
5,574 SMS messages labelled `ham` (not spam) or `spam`, of which 747 (13.4%) are spam.

The dataset is **not included** in this repo. Download it, unzip it, and place the file
`SMSSpamCollection` in a `data/` folder.

## Setup

    python -m venv .venv
    .venv\Scripts\activate
    pip install -r requirements.txt

Then open `spam_filter.ipynb`, select the `.venv` kernel, and use **Run All**.

## How it works

1. Load the messages and split them 80/20 into training and test sets (stratified, `random_state=42`).
2. Convert each message into numbers with `TfidfVectorizer`. The vocabulary (7,716 words) is learned
   from the training set only, so the test set stays unseen.
3. Train `MultinomialNB` on the training set.
4. Measure accuracy on both sets and compare with a baseline.

## Results

| | Accuracy |
|---|---|
| Baseline (always predict "not spam") | 86.6% |
| Train | 97.4% |
| Test | 96.2% |

The model beats the baseline by about 10 percentage points. Train and test accuracy are close,
so it is not overfitting; it generalizes well to messages it has never seen.

## Messages that fooled the model

| My message | Model said | Should be | Why it failed |
|---|---|---|---|
| Cl@im ur fr33 g1ft c@rd n0w, txt b@ck | ham | spam | **Spelling tricks.** "fr33", "g1ft" and "n0w" are not in the vocabulary learned from training, so they are ignored. Only ordinary-looking pieces are left. |
| F R E E  c a s h  p r i z e, t e x t  W I N  n o w | ham | spam | **Spaced-out letters.** The vectorizer ignores one-letter tokens, so every word disappears. The model sees an empty message and falls back to the most common class, ham. |
| Dear customer, your account statement is ready. Please confirm your details at our secure portal to avoid suspension. | ham | spam | **Polite, business-style phishing.** It avoids typical SMS spam words like "free", "win" and "prize". Words like "account", "details" and "please" mostly appear in normal messages in the training data. |
| Hi mum, I lost my phone, this is my new number. Can you send me some money today? I'll explain later | ham | spam | **A scam written like a family chat.** Every word is common in normal messages. The model counts words; it can't understand that asking for money from a "new number" is suspicious. |
| URGENT: call me now, the prize draw results are out and our team won free tickets | spam | ham | **A real message full of spammy words** ("urgent", "call", "now", "prize", "won", "free"). The model counts words but ignores context, so it can't tell that a friend is sharing good news. |

**A trick that did *not* work:** "This is not a free prize, just a reminder that our meeting is at 3"
was correctly classified as ham. Words like "reminder" and "meeting" outweighed "free" and "prize".

## What I learned

A model only sees the words it learned during training. Anything new, like misspellings or
spaced-out letters, is invisible to it, and that is exactly what spammers can exploit.
High accuracy is not the whole story either: the model counts words without understanding
their meaning, so tone, context and intent can fool it in both directions.