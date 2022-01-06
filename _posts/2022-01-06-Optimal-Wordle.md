---
layout: post
title:  "How To Play Perfect Wordle"
date:   2022-01-06
categories: jekyll update
---

# Intro

If your twitter feeds looks anything like mine, you are probably seeing 
a lot of posts like look something like this:

```
Wordle 201 5/6

⬛ 🟩 ⬛ ⬛ ⬛
⬛ 🟩 🟩 ⬛ ⬛
🟨 🟩 🟨 ⬛ ⬛
🟩 🟩 🟨 ⬛ ⬛
🟩 🟩 🟩 🟩 🟩
```

It's from this great daily game called <a href="https://www.powerlanguage.co.uk/wordle/" target="_blank">Wordle</a>. If you haven't play it, click the link and
play. It's very short and will help make this post make a lot more sense.

# What is the best start word?

You've probably realized that the word you start out with can make a huge difference
in how quickly you can solve the wordle puzzle. You probably found a word 
you like - but what is the BEST start word?

If you're not a nerd and just want the answer go here to the table at the 
end of this 
section, otherwise,
first we have to define what do we mean by the best....

It's easiest to start with an example:

Assume the hidden answer word is `T I G E R` and your start word is `E A R T H`.

The game will tell you that the hidden answer word:
- Has a `E` somewhere in the word, but in not as the first letter
- Does not have an `A`
- Does not have an `R`
- Has a `T` somwhere in the word, but not in the fourth spot
- Does not have an `H`

There are 12,972 total five letter words as described in the <a href=" https://drive.google.com/file/d/1oGDf1wjWp5RF_X9C7HoedhIWMh5uJs8s/view" target="_blank">scrabble 
dictionary</a>. But there are only 144 words that fit all of these criteria. 

The best guess will result in the fewest words left to choose from.

So obviously, we don't know what the hidden answer is, but what we can do is
say how many words are possible if the answer is `T I G E R`, 
and if is the answer is `W O R L D`,
and `A B O U T` and every single other possible five letter word. Then if we
take the average of every single possible answer, 
we can say how many words we *expect* to have left after we guess our 
start word.  
(just the plain average since every word is equally likely to be the answer)

Here is the distribution of how many words are left if you first guess 
`EARTH` across every single possible answer. You can look at the distributions 
any guess word [here]().

<img src="{{site.url}}/resources/guess_histogram.png" alt="hist">

Average number of words remaining after guessing `EARTH`: 819.1

If we measure this for all possible start words, we can say with absolute 
certainty (see complications below) which start word is best on expectation. 

Here are the top 15 best words

#### Complications

In this analysis we assume that all valid 5 letter words are equally likely to
be the answer. I think this is correct, given some recent odd answers like 
`REBUS` and `BANAL` but it could not be. All of this still works if the answers
are randomly selected by the probability of frequency of use, the math just 
gets a little trickier see the section "hat if the answers aren't selected randomly"

For a single guess, we need to evaluate how good that guess is across all 
possible words. And we need to evaluate all possible words. That's a LOT 
of computation. `12,972^2 = 168,272,784` to be exact. If I ran that on a 
single thread for every possibility it would take 9 days to run. I speed it up
by 
1. Parallelizing across my cores (thanks gnu parallel!)
1. Only considering words which have no double letters 
(easy to see why this is a bad strategy for a first guess)
1. Sampling all possible answers (50) first to get the top 100 possible best 
first guesses, then rerunning those on all possible answers.

## What is the worst start word?

## How To Play Perfectly

## What if the answers aren't selected randomly

If every answer is not equally likely, it would make sense then that every
word has a probability of being the answer with probability proportional 
to its frequency of use. 

This mainly changes two things: 
1. Instead of tracking the number of possible words. We need to track the 
entropy of the probability distribution of the remaining words. Entropy 
measures how much uncertainty there is in a probability distribution. For 
example, a uniform distribution between 0 and 10 has less entropy than a 
uniform distribution between 0 and 100. Similarly, a uniform distribution 
has more distribution than a "pointy" distribution 
(the shape of the distribution helps us guess what the outcome would be)
2. Instead of just taking the average entropy across all words, we need to 
take the expectation. Namely, the probability of the word being the answer
 times the entropy of the remaining words summed across all possible words.