---
title: A Review of Spelling Correction
date: '2017-04-16'
Author: Yang Song
thumbnailImagePosition: left
thumbnailImage: "img/spelling_correction.jpg"
---

A tutorial on Non-Word Spelling Correction and Real Word Spelling Correction
<!--more-->




*Note : The ideas and contents of this tutorial is from Stanford Professor Dan Jurafsky*

### Background
Recently, I am always interrupted by the annoying [Grammarly](https://www.grammarly.com/) ads on Youtube. In the same time, I can't help to watch and astonished by the powerful features in this playform, one of the feature is to correct misspelled words in your email, flags words in a email that may not be spelled correctly. Then I decided to learn the ideas behind it. Spelling correction is often considered from two perspectives. **Non-word spelling correction** is the detection and correction of spelling errors that result in non-words (like *speling* for *spelling*, *corection* for *correction*). By contrast, **real word spelling correction** is the task of detecting and correcting spelling errors even if they accidentally result in an actual word of English (I like to eat Cheescake Factory *desert*: which is suppose to be *dessert*)

- **Non-word spelling errors** are detected by looking for any word not found in a dictionary. For example, the word *speling* can't be found in a dictionary. To correct non-word spelling errors we need to generate **candidates** : real words that have a similar pattern to the error. Candidate corrections from the spelling error *speling* might include *speeding*, *spoiling*, *spiking*, *spending* or *spelling*. Then we rank the candidate using a **distance metric** between the source and the surface error.  We want the word *spelling* has the highest probability among all the candidates

- **Real-word spelling error** detection is much more difficult, since any word in the input text could be an error. However, we could still find the candidate of all words that occurs in a sentence and rank all the combinations to find the users original intentional one


### Methods
In order to learn the spelling correction, we need to introduce the **noisy channel model** first and show how to apply it to the task of detecing and correcting spelling errors.

The intuition of the **noisy channel** model is to treat the misspelled word as if a correctly spelled word had been ""distorted" by being passed through a noisy communication channel. This channel introduces "noise" in the form of substitution or other changes to the letters, making it hard to recognize the "true" word. Our goal is to build a model of the channel. Given this model, we then find the true word by passing every candidate word of the language through our model of the noisy channel and seeing which one comes the closest to the misspelled word.

The noisy channel is a kind of **Bayesian Inference**. For each given misspelled word *x*, Our job is to find one correct spelled word *w* with the highest possibility.

                            w_hat = argmaxP(w|x)

### A baby version of Word Spelling Corrector


#### Python Version

{{< codeblock "spell.py" "python" "http://underscorejs.org/#compact" "spell.py" >}}


import re
from collections import Counter

def words(text) : return re.findall(r'\w+', text.lower())

WORDS = Counter(words(open('big.txt').read()))

def P(word, N = sum(WORDS.values())):
    # Probability of the word
    return WORDS[word] * 1.0 / N

def correction(word):
    # Return the maximum prob word
    return max(candidate(word), key = P)

def candidate(word):
    return (known([word]) | known(edit1(word)) | known(edit2(word)) | {word})

def known(words):
    return set(w for w in words if w in WORDS)

def edit1(word):
    # All edit that are one edit away from word
    letters = "abcdefghijklmnopqrstuvwxyz"
    splits = [(word[:i], word[i:])for i in range(len(word) + 1)]
    deletes = {L + R[1:] for L, R in splits if R}
    inserts = {L + letter + R for L, R in splits for letter in letters}
    sub = {L + letter + R[1:] for L, R in splits if R for letter in letters}
    trans = {L + R[1] + R[0] + R[2:] for L, R in splits if len(R) > 1}
    return deletes | inserts | trans | sub | {word}

def edit2(word):
    return {e2 for e1 in edit1(word) for e2 in edit1(e1)}

{{< /codeblock >}}