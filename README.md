# Background
Word error rate (WER) is a commonly used metric for determining the efficacy of automatic speech recognition models. The simple formula for word error rate tallies substitutions, insertions,
and deletions and divides the total number of errors by the number of words in the reference transcription (S+D+I / N). This is computationally simple, but it fails to capture effective quality
of the predicted transcript because it treats all errors as equal. 

For example: given the reference sentence "I want to sleep," the prediction "I want sleep" (one deletion) and the prediction "I want to sheep" (one substitution) both have a WER of 0.25. However,
the first prediction has effectively the same meaning as the reference while the second is completely nonsensical. This motivates a need for a tanscription error metric which accounts for 
informational density of each word.

# This Experiment
This python notebook demonstrates an experimental "Information Weighted Word Error Rate" which weights traditional WER with semantic relevance of mispredicted words. 

## Word-wise weighting with LLM
After computing a word position aligned matrix of all substitutions, insertions, and deletions using the Levenshtein Distance Algorithm, the Gemini API is used to generate a weight vector 
for the semantic relevance of each word. This weight vector is multiplied with the SDI matrix and the IWWER is calculated as the sum of all nonzero elements / number of words in the reference
transcript.

This is not new research -- papers like [this one](https://ieeexplore.ieee.org/document/10818270) by Yutao Zhang and Jun Ai (2024) have explored this concept using more robust methods. 
I only made this as a lightweight exploration of the topic out of my own interest, but I'm happy for anyone to play around with it!

Both the unpredictability and response time of an LLM like gemini-2.5-flash (as used here) make this an unrealistic metric for use in practice, but those are areas I hope to explore further 
in the future.

## Sentence-wise weighting with embedding similarity (Added 9-30-25)
The main issues with word-wise LLM weighting come from the fact that LLM queries are slow and inconsistent. Both of these issues can be addresssed, albeit at the expense of granularity/explainability, by using a far more efficient and predictable text embedding model to compare semantic similarity of the entire prediction vs. the entire reference. This similarity value can be obtained through simple cosine similarity of the two embeddings, at which point a weighting factor can be determined by passing the similarity through an exponential mapping function. 
\[
w(x) = \alpha \, e^{-\beta \, \bigl(2(x - 0.5)\bigr)}
\]
Where alpha controls the maximum possible error factor (when two sentences are completely dissimilar) and beta controls the sharpness of the asymptote.

The mapping function, of form serves two purposes:
1. Google's open embedding model tends to classify most sentence pairs' similarity within the 0.5-1 range, so this must be normalized.
2. By adjusting alpha and beta, we can push the error weighting factor for semantically accurate predictions down to a very low value while pushing the weighting factor for highly innacurate predictions much higher. Finetuning these parameters lets us determine how much semantic similarity contributes to the final error value as opposed to traditional WER.

Of course, this options begs the question as to whether or not including traditional WER is even appropriate, since the semantic similarity factor on its own could provide a similar metric. However, incorporating WER may ground error values within a more consistent range than simply relying on direct embedding similarity (assuming the mapping function parameters are well tuned). I'm not sure how useful this approach is, to be honest, but I thought it was an interesting comparison with the LLM algorithm! Unlike the LLM method, this metric computes quickly enough to be feasibly used in training a model, so maybe more experiments can be done in the future.

# Usage
To use this notebook, a .env file of the format shown in .env-example must be included with your Gemini API key. Otherwise, running this notebook shouldn't require any special procedure.
