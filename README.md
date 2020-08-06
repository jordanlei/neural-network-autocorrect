# Correcting Autocorrect: In Pursuit of a Context-Based Word Processor
Contributors: Daniel Stekol, Jordan Lei
Special thanks to: TA Jeffrey Cheng and Prof. Konrad Kording

Using Neural Networks to perform context-based autocorrect.
Visit our [final project report here.](http://cis700dl.com/student_projects/leihao_5284918_76281416_Correcting_Autocorrect_Paper%20(1).pdf)

# Introduction
Essentially every modern device capable of typing includes some form of error correction - Microsoft
Word underlines grammatical errors, and smartphones often shamelessly substitute the word you typed
for the word they think you meant, leading to the widely-known Cupertino Effect (known in the NLP
community as “damn you, autocorrect!”). Many such systems rely on metrics such as edit-distance, or
perhaps even account for keyboard layout, but few of them are actually sensitive to the content of the
entire sentence.

For instance, consider the following sentence: 
>After obtaining my diploma, I plan on pursuing graduate students in Computer Science.

A human reading this sentence would stop, spit out their drink, peruse the sentence a second time, and
most likely conclude that the writer meant "graduate students", not that they would take up stalking as a
hobby. A word processor, however, would almost certainly fail to flag this as an error, since the bigram
"graduate students" is quite common, and only the presence of "pursue" suggests that "students" might be
the wrong choice. Note that the word "students" is quite lexically similar to "studies", and it is therefore
conceivable that a small spelling mistake could lead to an auto-correction that results in this unfortunate
phrase.

The goal of this project, therefore, is to create a model that can detect such abnormalities in sentences,
even when the substituted word is reasonably similar to the original word. To this end, we have created a
dataset by perturbing an existing corpus, trained both traditional and neural machine learning models, and
evaluated their performance and relative advantages. Ultimately, we have found that although this
identification task is quite difficult to learn for all the models, deep-learning models are able to both
significantly improve over random performance, and also substantially outperform the traditional machine
learning algorithms examined.

# The Dataset
We took an existing collection of sentences and corrupted them, by inserting words that didn’t belong in a
given sentence with words that were 3 edits away (for example, “through” and “rouge”). In one version,
we created a dataset where replacement words were the same part-of-speech as the word they were
replacing (the so-called filtered dataset); in another, any word could be replaced by any other, so long as
they were the appropriate edit distance away (the unfiltered dataset).
In each sentence, every word was transformed into a vector representation using PyMagnitude, a library
that converts words into word-embeddings of length 300. Thus, each sentence was represented by a list of
length-300 vectors.

# The Models
The goal of each of our models is to take a sentence (a list of word embeddings) and have it output a 1 for
each word that belongs, and a 0 for each word that doesn’t.

We wanted a model to compare our performance to, so we chose a baseline model that would be a good
benchmark for what we were getting ourselves into. The simplest way to test the difficulty of this dataset
would be to take each sentence and average the word-vectors, then concatenate this vector with the
existing word embedding. Using this new vector, we fed the data through a logistic regression to see if it
could learn when a word did not belong.

Eagle-eyed readers will recognize that this model has significant drawbacks. The most significant
drawback is that by collapsing the entire sentence into a single vector, we lose all sequential information,
which is necessary to determine whether a word belongs in a given context. In the example we described
before with the graduate students vs. studies, we relied on the fact that pursue came before graduate
students to recognize something was amiss. Clearly, we needed a model that was capable of recognizing
the importance of sequential ordering in sentences and was able to remember relevant details. Cue the
LSTM.

Our LSTM models (vanilla and bi-directional) helped fulfill our need for a model that could handle both
word-embeddings and the sequential ordering of words. LSTMs work well when inputs have both long
and short-term patterns. Our bidirectional LSTM also had the added benefit of capturing relationships
that occured after the words in question. In our example “... pursuing graduate [students/studies] in
computer science”, the phrase “in computer science” (which comes after the erroneous word) may be of
use in determining whether or not a given word belongs.

# Conclusions
Our baseline model was used as a measure of how difficult the dataset was to learn. When fully trained,
the logistic regression got to about 8% accuracy. Not bad, but not great either. We found that instead of
learning accurately, it was just predicting that everything was correct! This indicated three things: (1)
logistic regression is not a sufficiently powerful tool to perform this task, (2) this task is pretty difficult,
and (3) perhaps we should rethink our loss function to prevent the classifier from always predicting 1.
Our vanilla LSTM performed much better, achieving about 18% on the filtered dataset. As a response to
observation (3), we re-trained our model on a weighted cross-entropy loss function rather than a simple
binary cross-entropy. After training the LSTM on the augmented dataset and weighted loss, we found that
the accuracy improved to 22%. We had similar findings for our bidirectional LSTM, which had accuracy
of 24% and later 29% with the weighted loss function.

We found that in all models, the accuracy was much better on the dataset filtered by part-of-speech than
the unfiltered dataset. While this may seem counterintuitive (it should be easier to identify a word that is
out of context by its part of speech alone in the unfiltered dataset), it could be the case that corrupting a
word in an unfiltered sense makes the entire sentence nonsensical, making it hard to identify which word
is the culprit.

We also found that in many cases, the models had low confidence across the board. When using “best guess accuracy”, where we had the model pick the “most wrong” word in a sentence even if it didn’t
designate a score below 0.5, we found that the models significantly improved. This indicates that our
models were able to pinpoint words that were less likely to belong, but did not have the confidence
threshold to flag them as erroneous. 
