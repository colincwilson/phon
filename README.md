
<!-- README.md is generated from README.Rmd. Please edit that file -->

# phon

[![Travis build
status](https://travis-ci.org/coolbutuseless/phon.svg?branch=master)](https://travis-ci.org/coolbutuseless/phon)[![codecov](https://codecov.io/gh/coolbutuseless/phon/branch/master/graph/badge.svg)](https://codecov.io/gh/coolbutuseless/phon)
[![Project Status: Active – The project has reached a stable, usable
state and is being actively
developed.](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active)

The goal of `phon` is to make available the CMU Pronouncing Dictionary
([**cmudict**](http://www.speech.cs.cmu.edu/cgi-bin/cmudict)) in an R
friendly format, and to collect some tools which use the pronunciation
information.

The CMU Pronouncing Dictionary includes pronunciations for 130,000
words. By matching the phonemes between words, this lets us find:

  - homophones e.g. `steak` and `stake`
  - rhymes e.g. `marry` and `actuary`
  - similar sounding words e.g. `statistics` and `stochastics`
  - words which contain the pronunciation of other words,
    e.g. `bathroom` contains the pronunciation of `threw`
  - syllable counts

This is a companion package to the [syn](http://syn.njtierney.com/)
package in that `syn` finds related words based upon meanings, while
`phon` finds related words based upon pronunciation.

## Installation

You can install `phon` from github with:

``` r
devtools::install_github("coolbutuseless/phon")
```

## Phonemes

Phonemes are the sounds which make up a word.

The phonetic encoding in `phon` come from the CMU Pronouncing Dictionary
([**cmudict**](http://www.speech.cs.cmu.edu/cgi-bin/cmudict)) which
encodes words using [ARPABET](https://en.wikipedia.org/wiki/ARPABET).

``` r
phon::phonemes("cellar")
#> [[1]]
#> [1] "S"   "EH1" "L"   "ER0"
```

Since some words have mutliple pronunciations, the results of
`phon::phonemes()` is always returned as a list, e.g. *carry* has two
slightly different pronunciations.

``` r
phon::phonemes("carry")
#> [[1]]
#> [1] "K"   "AE1" "R"   "IY0"
#> 
#> [[2]]
#> [1] "K"   "EH1" "R"   "IY0"
```

ARPABET phonetic encoding includes stress markers as suffixes to vowel
phonemes. The markers are:

0.  No stress
1.  Primary stress
2.  Secondary stress

You can ask for phonemes without the stress markers, e.g.

``` r
phon::phonemes("fantastic")
#> [[1]]
#> [1] "F"   "AE0" "N"   "T"   "AE1" "S"   "T"   "IH0" "K"
phon::phonemes("fantastic", keep_stresses = FALSE)
#> [[1]]
#> [1] "F"  "AE" "N"  "T"  "AE" "S"  "T"  "IH" "K"
```

## Syllables

The number of syllables in a word is the count of the number of phonemes
with stress markers in the word.

This is pre-calculated and available through the `phon::syllables()`
function.

``` r
phon::syllables("average")
#> [1] 3
phon::syllables("antidisestablishmentarianism")
#> [1] 12
```

## Matching Phonemes within Words

`phon` allows you to search for the sound of one word within another.

In the following example, `phon::contains_phonemes()` finds all the
words that include the pronunciation of *“through”* within their
pronunciation.

``` r
through <- phon::phonemes("through")[[1]]
through
#> [1] "TH"  "R"   "UW1"
```

``` r
phon::contains_phonemes(through)
#>  [1] "bathroom"      "bathrooms"     "bathrooms"     "breakthrough" 
#>  [5] "breakthroughs" "drive-thru"    "drive-thrus"   "overthrew"    
#>  [9] "threw"         "throop"        "through"       "throughout"   
#> [13] "throughput"    "throughs"      "throughway"    "thru"         
#> [17] "thruway"
```

Use the `keep_stresses` argument to match with/without the stresses
included (default is to ignore the stresses).

## Homophones

Homophones are words with the same pronunciation but different spelling.

``` r
phon::homophones("steak")
#> [1] "stake"
phon::homophones("carry")
#>  [1] "carey"  "carie"  "carrey" "carrie" "cary"   "kairey" "kari"  
#>  [8] "karry"  "kary"   "kerrey" "kerri"  "kerry"
```

## Rhymes

To find rhymes, `phon` compares trailing phonemes. If the phonemes at
the end of a word in the dictionary match those at the end of the given
word, then they rhyme.

The rhymes are returned in multiple vectors:

  - Words with the most matching trailing phonemes are returned first.
  - Subsequent vectors have fewer matching trailing phonemes.
  - The names of the list are the number of trailing phonemes which
    match.

<!-- end list -->

``` r
phon::rhymes("drudgery")
```

    $`3`
     [1] "challengery"  "forgery"      "gingery"      "injury"       "margery"     
     [6] "marjorie"     "marjory"      "menagerie"    "neurosurgery" "perjury"     
    [11] "surgery"     
    
    $`2`
      [1] "acary"           "accessory"       "adoree"          "adultery"       
      [5] "advisory"        "alimentary"      "alphandery"      "ambery"         
      [9] .... (results trimmed)

In the above example:

  - The phonemes for “drudgery” are “D R AH1 JH ER0 IY0”
  - In the first vector, the words match the *-gery* sound, i.e the last
    3 phonemes.
  - In the second vector the words only match the *-ery* sound, i.e. the
    last 2 phonemes.

## Similar sounding words

Similar sounding words are found by comparing words with the same number
of phonemes but with a number of mismatches allowed.

``` r
phon::similar("statistics", phoneme_mismatches = 5)
#>  [1] "anaesthetics" "anesthetics"  "centronics"   "gymnastics"  
#>  [5] "heuristics"   "onomastics"   "scientific's" "scientifics" 
#>  [9] "statistics'"  "stochastics"  "subsistence"  "synbiotics"
```

## Accessing the raw data

The actual *cmudict* data is stored within the package in a number of
pre-processed forms. This reduces overall memory footprint, and enables
faster lookups of information about each word.

If you’d like to take this data external to `phon` and do something with
it, then there are two functions for exporting the data:
`create_cmudict_as_list()` and `create_cmudict_as_data_frame()`

To get a copy of the data as a list:

``` r
head(phon::create_cmudict_as_list())
#> $`!exclamation-point`
#>  [1] "EH2" "K"   "S"   "K"   "L"   "AH0" "M"   "EY1" "SH"  "AH0" "N"  
#> [12] "P"   "OY2" "N"   "T"  
#> 
#> $`"close-quote`
#> [1] "K"   "L"   "OW1" "Z"   "K"   "W"   "OW1" "T"  
#> 
#> $`"double-quote`
#> [1] "D"   "AH1" "B"   "AH0" "L"   "K"   "W"   "OW1" "T"  
#> 
#> $`"end-of-quote`
#> [1] "EH1" "N"   "D"   "AH0" "V"   "K"   "W"   "OW1" "T"  
#> 
#> $`"end-quote`
#> [1] "EH1" "N"   "D"   "K"   "W"   "OW1" "T"  
#> 
#> $`"in-quotes`
#> [1] "IH1" "N"   "K"   "W"   "OW1" "T"   "S"
```

To get a copy of the data as a data.frame:

``` r
phon::create_cmudict_as_data_frame()
#> # A tibble: 133,854 x 4
#>    word         phonemes                phonemes_sans_stresses   syllables
#>    <chr>        <chr>                   <chr>                        <int>
#>  1 !exclamatio… EH2 K S K L AH0 M EY1 … EH K S K L AH M EY SH A…         5
#>  2 "\"close-qu… K L OW1 Z K W OW1 T     K L OW Z K W OW T                2
#>  3 "\"double-q… D AH1 B AH0 L K W OW1 T D AH B AH L K W OW T             3
#>  4 "\"end-of-q… EH1 N D AH0 V K W OW1 T EH N D AH V K W OW T             3
#>  5 "\"end-quot… EH1 N D K W OW1 T       EH N D K W OW T                  2
#>  6 "\"in-quote… IH1 N K W OW1 T S       IH N K W OW T S                  2
#>  7 "\"quote"    K W OW1 T               K W OW T                         1
#>  8 "\"unquote"  AH1 N K W OW1 T         AH N K W OW T                    2
#>  9 #hash-mark   HH AE1 M AA2 R K        HH AE M AA R K                   2
#> 10 #pound-sign  P AW1 N D S AY2 N       P AW N D S AY N                  2
#> # ... with 133,844 more rows
```

## CMU Pronouncing Dictionary

This package relies on the great Pronouncing Dictionary by CMU.

### CMU Pronouncing Dictionary Copyright notice

    CMUdict  --  Major Version: 0.07
    
    $HeadURL$
    $Date::                                                   $:
    $Id::                                                     $:
    $Rev::                                                    $:
    $Author::                                                 $:
    
    
    ========================================================================
    Copyright (C) 1993-2015 Carnegie Mellon University. All rights reserved.
    
    Redistribution and use in source and binary forms, with or without
    modification, are permitted provided that the following conditions
    are met:
    
    1. Redistributions of source code must retain the above copyright
       notice, this list of conditions and the following disclaimer.
       The contents of this file are deemed to be source code.
    
    2. Redistributions in binary form must reproduce the above copyright
       notice, this list of conditions and the following disclaimer in
       the documentation and/or other materials provided with the
       distribution.
    
    This work was supported in part by funding from the Defense Advanced
    Research Projects Agency, the Office of Naval Research and the National
    Science Foundation of the United States of America, and by member
    companies of the Carnegie Mellon Sphinx Speech Consortium. We acknowledge
    the contributions of many volunteers to the expansion and improvement of
    this dictionary.
    
    THIS SOFTWARE IS PROVIDED BY CARNEGIE MELLON UNIVERSITY ``AS IS'' AND
    ANY EXPRESSED OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO,
    THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
    PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL CARNEGIE MELLON UNIVERSITY
    NOR ITS EMPLOYEES BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
    SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT
    LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE,
    DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY
    THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
    (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
    OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
    
    ========================================================================
