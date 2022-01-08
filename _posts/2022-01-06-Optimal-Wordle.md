---
layout: post
title:  "How To Play Perfect Wordle"
date:   2022-01-06
categories: jekyll update
---

# Intro

If your twitter feed looks anything like mine, you are probably seeing 
a lot of posts that look something like this:

```
Wordle 201 5/6

⬛ 🟩 ⬛ ⬛ ⬛
⬛ 🟩 🟩 ⬛ ⬛
🟨 🟩 🟨 ⬛ ⬛
🟩 🟩 🟨 ⬛ ⬛
🟩 🟩 🟩 🟩 🟩
```

It's from a great daily game called <a href="https://www.powerlanguage.co.uk/wordle/" target="_blank">Wordle</a>. If you haven't played it, click the link and
play. It's very short and will help make this post make a lot more sense.

# Solver

[This lil app](https://share.streamlit.io/tjboller/wordle/main/visualizations.py)
I made has a tool to help you solve the puzzles optimally.

# What is the best start word?

You've probably realized that the word you start out with can make a huge difference
in how quickly you can solve the puzzle. You might have found a word 
you like - but what is the BEST start word?

The answer is `L A R E S`

How did we get that? First we have to define what do we mean by the best.

It's easiest to start with an example:

Assume the hidden answer word is `T I G E R` and your start word is `E A R T H`.

The game will tell you that the hidden answer word:
- Has a `E` somewhere in the word, but in not as the first letter
- Does not have an `A`
- Does not have an `R`
- Has a `T` somewhere in the word, but not in the fourth spot
- Does not have an `H`

There are 12,972 total five letter words as described in the <a href=" https://drive.google.com/file/d/1oGDf1wjWp5RF_X9C7HoedhIWMh5uJs8s/view" target="_blank">scrabble 
dictionary</a>. But there are only 144 words that fit all of these criteria. 

The best guess will result in the fewest words left to choose from.

So obviously, we don't know what the hidden answer is, but what we can do is
say how many words are possible after our first guess 
if the answer is `T I G E R`, 
and follow the same process if is the answer is `W O R L D`,
and `A B O U T` and every single other possible five letter word. So if we
take the average of every single possible answer, 
we can say how many words we *expect* to have left after we guess our 
start word.  
(It's the plain average since every word is equally likely to be the answer)

Here is the distribution of how many words are left if you first guess 
`EARTH` across every single possible answer. You can look at the distributions 
any guess word [here](https://share.streamlit.io/tjboller/wordle/main/visualizations.py).

<img src="{{site.url}}/resources/guess_histogram.png" alt="hist">

Average number of words remaining after guessing `EARTH`: 670.5

If we measure this for all possible start words, we can say with absolute 
certainty (see complications below) which start word is best on expectation. 

Here are the top 15 best words.

<table border="1" class="dataframe"><thead><tr style="text-align: right;"><th>Guess</th><th>Average Possible Words After Guess</th></tr></thead><tbody><tr><td>lares</td><td>288.738205</td></tr><tr><td>rales</td><td>292.106075</td></tr><tr><td>tares</td><td>302.491674</td></tr><tr><td>soare</td><td>303.830866</td></tr><tr><td>reais</td><td>304.761024</td></tr><tr><td>nares</td><td>305.548720</td></tr><tr><td>aeros</td><td>309.733426</td></tr><tr><td>rates</td><td>311.355381</td></tr><tr><td>arles</td><td>314.733888</td></tr><tr><td>serai</td><td>315.133364</td></tr><tr><td>saner</td><td>317.736047</td></tr><tr><td>tales</td><td>325.594820</td></tr><tr><td>aloes</td><td>326.700123</td></tr><tr><td>reals</td><td>329.109004</td></tr><tr><td>lanes</td><td>330.211224</td></tr></tbody></table>

Here are the 5 worst words

<table border="1" class="dataframe">  <thead>    <tr style="text-align: right;">      <th>word</th>      <th>count</th>    </tr>  </thead>  <tbody>    <tr>      <td>hudud</td>      <td>4454.262411</td>    </tr>    <tr>      <td>phpht</td>      <td>4458.732809</td>    </tr>    <tr>      <td>gynny</td>      <td>4677.415665</td>    </tr>    <tr>      <td>qajaq</td>      <td>4960.594665</td>    </tr>    <tr>      <td>fuzzy</td>      <td>5041.903639</td>    </tr>    <tr>      <td>cocco</td>      <td>5046.276133</td>    </tr>    <tr>      <td>hyphy</td>      <td>5237.960839</td>    </tr>    <tr>      <td>immix</td>      <td>5248.980419</td>    </tr>    <tr>      <td>gyppy</td>      <td>5384.723558</td>    </tr>    <tr>      <td>fuffy</td>      <td>5396.426303</td>    </tr>  </tbody></table>

#### Complications

In this analysis we assume that all valid 5 letter words are equally likely to
be the answer. I think this is correct, given some recent odd answers like 
`REBUS` and `BANAL` but it could not be. All of this still works if the answers
are randomly selected by the probability of frequency of use, the math just 
gets a little trickier see the section "what if the answers aren't selected randomly"

Another complication is that for a single guess, 
we need to evaluate how good that guess is across all 
possible words. And we need to evaluate all possible words. That's a LOT 
of computation. `12,972^2 = 168,272,784` to be exact. If I ran that on a 
single thread for every possibility it would take many many days to run. 
I speed it up by 
1. Parallelizing across my cores
1. Randomly sampling 100 possible answers across first to get the top 
250 possible best first guesses, then rerunning those on all possible answers.

Also, this is using a greedy approach. It might not be globally optimal. The 
global optimization problem is completely computational intractable.

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
has more distribution than a "pointy" distribution over the same interval
(the shape of the distribution helps us guess what the outcome would be)
2. Instead of just taking the average entropy across all words, we need to 
take the expectation. Namely, the probability of the word being the answer
 times the entropy of the remaining words summed across all possible words.
 
## Playing Optimally

If we play optimally what is the distribution of the number of turns it takes
to win?

TO COME 
 
## Reverse Engineering Heuristics 

TO COME