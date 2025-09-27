# Background
Word error rate (WER) is a commonly used metric for determining the efficacy of automatic speech recognition models. The simple formula for word error rate tallies substitutions, insertions,
and deletions and divides the total number of errors by the number of words in the reference transcription (S+D+I / N). This is computationally simple, but it fails to capture effective quality
of the predicted transcript because it treats all errors as equal. 

For example: given the reference sentence "I want to sleep," the prediction "I want sleep" (one deletion) and the prediction "I want to sheep" (one substitution) both have a WER of 0.25. However,
the first prediction has effectively the same meaning as the reference while the second is completely nonsensical. This motivates a need for a tanscription error metric which accounts for 
informational density of each word.

# This Experiment
This python notebook demonstrates an experimental "Information Weighted Word Error Rate" which weights traditional WER with semantic relevance of mispredicted words. 

After computing a word position aligned matrix of all substitutions, insertions, and deletions using the Levenshtein Distance Algorithm, the Gemini API is used to generate a weight vector 
for the semantic relevance of each word. This weight vector is multiplied with the SDI matrix and the IWWER is calculated as the sum of all nonzero elements / number of words in the reference
transcript.

This project is not new research -- papers like [this one](https://ieeexplore.ieee.org/document/10818270) by Yutao Zhang and Jun Ai (2024) have explored this concept using more robust methods. 
I only made this as a lightweight exploration of the topic out of my own interest, but I'm happy for anyone to play around with it!

Both the unpredictability and response time of an LLM like gemini-2.5-flash (as used here) make this an unrealistic metric for use in practice, but those are areas I hope to explore further 
in the future.
