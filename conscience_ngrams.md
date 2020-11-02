
# References {-}


:::{#refs}
:::

\appendix
# Text Analysis

\fancyhf{}
\fancyhead[RO,LE]{\thepage}
\fancyhead[RE,LO]{\leftmark}

This appendix presents the code for the distant reading section of the Experience chapter, along with commentary about the methods in use.

The code is implemented in R. Two systems dominate the academic data analysis landscape: Python and R. Both have the virtue of being free and open source. In recent years there have been many packages (self-contained chunks of code providing methods and functions) developed for use in digital scholarship. This workbook employs several of these packages.^[@Wickham2019 and @Silge2016 get the most use.] This appendix reproduces the code required to analyze Google's dataset. The Hathi Trust contains a similar dataset, and I have conducted a similar analysis on that data. The difference is that the Hathi Trust data has to be extracted via their approved methods. The results of that analysis do not substantially differ from the Google data, but the data-collecting code is much more complex.

The relevant code, along with all the other analyses, is available online at  [https://github.com/adgarland/conscience](https://github.com/adgarland/conscience).

<!-- -->

## Google corpus

Google has datasets of ngrams (up to 5grams) available freely. The datasets are organized by the first two letters, so we can find a lot of "conscience" ngrams by downloading the set of 5grams that start with "co". This is a huge data set. When uncompressed it is about 42.6Gb. Since R holds all of its working data in RAM, we would need a computer with well more than 45Gb of RAM to handle a file this size in R directly.

There are some ways around these very large files. R has packages to chunk large files on the disk so that various operations can load the necessary data in pieces, do the operations, and then return the results. But in our case, we can use a simpler, more direct method. The dataset is all the ngrams starting with "co", and we might reasonably assume that the vast majority of them do not even *include* the word "conscience", much less start with it. Happily, there is a common command-line utility available in Unix systems by which we can filter out the ngrams that do not include "conscience".^[Windows users can do something similar with PowerShell, and Windows 10 users can install a Linux subsystem that gives access to Unix commands, including grep. This is how I performed the task. The following command tells the computer to look line-by-line for a string of letters that looks like "con" plus something plus "cienc", without respecting the letter case. The wildcard where the S should be covers transcriptions of the archaic "s", which often gets misread as "f".]

~~~~~~~~

grep -i "con.cienc" input_file.txt >> output_file.txt

~~~~~~~~

Even better, Google has tagged much of its corpus by part of speech. So "conscience" often appears as "conscience_NOUN", and other words in the ngrams are similarly tagged. This will be very useful, for it will allow us to look at verbs and adjectives, which is what we wanted anyway.

One disadvantage of this dataset is that it will not have the words immediately *preceding* "conscience." We will have to get those a different way. In particular, we would be looking for a larger dataset like the one used above. Google's ngram viewer presents only the top ten wildcard results or so, which means any rarer terms will not be available.

It is common in text analysis to weed out very common words, usually because they are not informative. One would expect that if we could get every word that precedes "conscience" in some huge corpus, we would find that the most frequent result is "the". (Some separate analysis gives credence to this guess.)

### Some examples

After cleaning, the dataset has 1,663,602 lines. If we filter out everything that doesn't start with "conscience", we get 36,136 lines. These numbers will be a useful source for comparison later, since the various counts can be understood in reference to them.

When we look for the words immediately following "conscience," we get 800 different items. And unsurprisingly, the most common words are not very helpful.

We will also strip out anything that uses punctuation or digits. Sometimes the text files have page numbers or punctuation after our word. It might be moderately interesting to see how many times "conscience" ends a sentence, but there isn't likely to much gain from it. So we will take those out.

The next steps involve cleaning up the data. We have a list of words, or word-like character strings. Now we need to see what they actually say.

First, we'll try to remove uninformative words. It is common in ngram analysis to remove "stop words"--those very common words that make the grammar work but don't add much meaning. Let's try that. The set here comes from the *tokenizers* package,^[@Mullen2018] and matches a common dataset used in other applications. Here is an example of stop words that we'll remove.

<div class="kable-table">

|words     |
|:---------|
|can't     |j         |indeed    |her       |
|outside   |lately    |certainly |an        |
|same      |young     |newer     |beings    |
|a         |overall   |there's   |your      |
|everyone  |under     |que       |cause     |

</div>


Second, and more interestingly, we can trim the words down to their stems. For example, we might expect to see separate entries for "conscience accused" and "conscience accuses". These are clearly attributing the same action to conscience, and so it will be useful to drop the last letter and treat these as one. The *tokenizers* package has a command to do just this, so we'll apply it next. We will look for all of the words following conscience, and list the most frequent *stems* along with their counts.

<div class="kable-table">

word2           n
---------  ------
sake        68261
i           35416
void        30858
tell        28793
stricken    23805
told        18799
smote       11342
freedom      8832
religion     8120
sear         6272
troubl       6110
bear         5978
prick        5826
make         5152
began        4870

</div>

![](figures/word_stems-1.png)<!-- -->

This is a useful list. Two of the entries are different tenses of "tell", which supports the intuition that conscience has a "voice". The most common word is "sake", and usually this word immediately follows the possessive form of conscience. The locution "conscience' sake" is a biblical one. It comes from Romans 13, where Christians are told to obey the civil authorities "for conscience' sake," and from 1 Corinthians 10, where the question is about meat offered to idols. If the New Testament is a strong influence on the way we talk about conscience, these sorts of distinctive phrases would be what we should expect to see.

The second most common is "I". I will speculate that this is because of the way tokenization works. "I" is a common first word for a sentence, and so it could be that when "conscience" ends a sentence, the next word is often "I". This would still be an interesting result, for it illustrates that when people talk about conscience, they are typically talking about themselves.

The fourth most common word is "void," and I think we can see another biblical allusion. Let's look at all the ngrams that use this word.

<div class="kable-table">

ngram                                      n
------------------------------------  ------
conscience void of offence             10671
conscience void of offence towards      6567
conscience void of offence toward       5212
conscience void of offense toward       1884
conscience void of offense              1422
conscience void of offence both         1164
conscience void of offence to            820
conscience void of offence before        574
conscience void of offense towards       562
conscience void of reproach              550
consciences void of offence towards      467
conscience void of offence in            450
consciences void of offence              373
conscience is void of offence            275
conscience be void of reproach           218

</div>

This is pretty clearly a quotation from Acts 24:16 (KJV), where Paul is defending himself before Felix, the Roman governor. "And herein do I exercise myself, to have always a conscience void of offence toward God, and toward men."

Two other words are "stricken" and "smote". These are different forms of the same word too. Let's look at "smote", since we saw that word used of King David.

<div class="kable-table">

ngram                                  n
---------------------------------  -----
conscience smote him                2578
conscience smote her                 972
conscience smote him and             820
conscience smote me                  740
conscience smote him for the         400
conscience smote him as he           348
conscience smote him that he         342
conscience smote me and              340
conscience smote me for having       300
conscience of the renegade smote     282
conscience smote him and he          260
conscience smote her for gazing      256
conscience smote him for his         254
conscience smote her and             246
conscience smote him at the          246

</div>

This result is less obviously a biblical quotation. Result 1 could include results 3, 5, 7, and 8.^[The way ngram tokenization occurs basically ensures that these four line are included in the first result, but some quick arithmetic shows that any additional results will have low counts. This property of ngram analysis is well-known and one of the reasons for caution against overinterpreting the results.] The evidence for the biblical link is less clear, though the basic locution is still very common.

An aside on the distribution of terms: Notice that these terms, even after being filtered, have a curved distribution. This distribution of words in a vocabulary is a common characteristic of natural language called Zipf's Law. The frequency of a word is inversely proportional to its rank in a frequency table.

![](figures/plot_next_words-1.png)<!-- -->

For completeness, let's see what came after "and" and "of" in the very common n-grams.


```r
and_words <- unique_clean %>% filter(word(ngram, 2) == "and") %>%  
    mutate(word3 = str_to_lower(word(ngram, 3))) %>%  
    anti_join(my_stop_words, by = c(word3 = "word")) %>%  
    group_by(word3) %>%  
    summarize(matches = sum(matches)) %>%  
    arrange(desc(matches))

of_words <- unique_clean %>% filter(word(ngram, 2) == "of") %>%  
    mutate(word3 = word(ngram, 3)) %>%  
    group_by(word3) %>%  
    summarize(matches = sum(matches)) %>%  
    arrange(desc(matches))

next_of_words <- unique_clean %>%  
    filter(word(ngram, 2) == "of" &  
      word(ngram, 3) %in% c("the", "a")) %>%  
    mutate(word3 = word(ngram, 3), word4 = word(ngram, 4)) %>%  
    group_by(word3, word4) %>%  
    summarize(matches = sum(matches)) %>%  
    arrange(desc(matches))
```


When we filtered out the stop words we dropped a number of common verbs. Helping verbs in particular will be notable, for though they themselves are too common to be instructive, they link "conscience" to some other word that may be more informative.


![](figures/get_third_words-1.png)<!-- -->

We can also look at the words that follow "be*". Then we'll look at the words that follow tenses of "is".


![](figures/linking_verbs-1.png)<!-- -->

Next we'll try to borrow from Google's part of speech tagging. First, we'll look at adjectives following a "be" verb.


<div class="kable-table">

word          pos         n
------------  -----  ------
conscience    NOUN    18734
consciences   NOUN     1521
god           NOUN      213
man           NOUN      207
men           NOUN      205
religion      NOUN      171
sense         NOUN      165
clear         ADJ       159
people        NOUN      151
sake          NOUN      140
good          ADJ       129
tells         VERB      121
freedom       NOUN      113
world         NOUN      111
heart         NOUN      108

</div>


![](figures/extended_pos-1.png)<!-- -->



![](figures/extended_pos-2.png)<!-- -->


A special case of the helping verbs is also worth noting. An ngram might go, "conscience did not ...". We would be interested in whatever goes in the elipsis.

<div class="kable-table">

next_word    matches
----------  --------
permit         16642
suffer          5814
reproach        3132
approve         2772
trouble         1902
accuse          1850
bother          1312
give            1032
bear             536
condemn          518
fail             512
assent           486
left             380
yield            372
prick            350

</div>

![](figures/negations-1.png)<!-- -->

In general, this large-scale analysis reveals patterns of usage around the word "conscience" that give evidence for the claim that there is a folk conception of conscience in regular circulation, and that it has roughly the features I have attributed to it.
