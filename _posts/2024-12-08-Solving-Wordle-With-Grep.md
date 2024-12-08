---
layout: post
share: true
title: Solving Wordle With Grep
date: 2024-12-08 00:00:00 -0400
filename: _posts/2024-12-08-Solving-Wordle-With-Grep
tags:
  - Wordle
  - Grep
  - Regex
  - Word-games
---

[Grep](https://en.wikipedia.org/wiki/Grep) is a command line tool on Unix-like systems for searching plaintext datasets for lines that match a [regular expression](https://en.wikipedia.org/wiki/Regular_expression). Grep has many uses such as finding files, email addresses, filtering text and basic text editing but it can be daunting to learn about regex when starting out. This post aims to provide a fun way to solve Wordle puzzles and also get better at regex at the same time.

## Why solve Wordle puzzles with grep?

Wordle puzzles are perfectly suited to being solved with regex as the solution set is relatively small ([the NYT Wordle bot](https://www.nytimes.com/2022/08/17/upshot/wordle-wordlebot-new.html) recognises about 3,150 5-letter words as viable Wordle solutions), meaning that we can search through the word list quickly on almost any hardware. Some users find the official Wordle bot opaque and frustrating and using regex is purely logic-based with the output available immediately. The challenge of solving the puzzle is also retained as there is no guarantee that the regex-based approach will be successful.

## How to Start Solving Wordle puzzles with grep?

In order to get started, we first need to make sure grep is installed - it should be pre-installed on most versions of Linux. Typing the following should indicate what version of grep is installed:

```bash
grep -V
```

If Grep isn't installed should be able to install it with the following command on most versions of Linux:

```bash
sudo apt install grep
```

If you are using Windows, you may want to look at installing [WSL](https://learn.microsoft.com/en-us/windows/wsl/install) firstly and then installing grep afterwards.

We also need to get a word list - I would suggest downloading a copy of the official wordlist from [here](https://wordletools.azurewebsites.net/weightedbottles). Make sure to remove the word weightings as we don't need them. We can strip these by redirecting the following grep output to a new file - make sure to first manually remove any headings or other unnecessary text.

```bash
grep -o "[A-Z]*" "raw_word_list.txt" > word_list.txt
```

Lets break the first part of this command down to get an idea how grep works:

```bash
grep -o
```

The `-o` here is a flag which tells grep we are only interested in seeing the output that matches the result of our regular expression. The regular expression itself `[A-Z]*` is made up of two parts - `[A-Z]` specifies that we are only interested in letters in the range `A-Z` (basically all capital letters in the alphabet) and the second part `*` is a quantifier expressing that we are interested in matching any number of the previous token (all capital letters) from 0 to unlimited times. This is a "greedy" search meaning that grep will try to return as many letters as possible.

Redirecting the output of this command to a new file leaves us with just the list of 5-letter words that we require.

## Regex 101

Lets test out grep by searching for a common 5-letter word that we know will be in our word list:

```bash
grep -i about "Wordle Words.txt"
```

This search should return the word "ABOUT" in the console if everything is working correctly. We use the `-i` flag for a case-insensitive search.

Essentially how grep works is by iterating through the text file character by character with a cursor. If the first letter "A" is matched, the cursor moves on to the next character, if "B" is matched it moves on to next and so on until it has matched the entire word which is returned as the result. This is the default behaviour for grep.

For the purposes of solving Wordle, we will be using a special type of search called a "lookahead assertion" which is where the cursor looks ahead at that point and checks if the following letters match the expression; however unlike the default behaviour the cursor position doesn't advance. If the assertion fails at that point no match is returned, however if the assertion is successful grep will try and match the next character or assertion from the same point. This behaviour means that we can test numerous conditions in a row without the cursor advancing.

The syntax for a positive lookahead assertion in grep is `(?=)`. We also need to use the `-P` flag which indicates that we are using a Perl regular expression which extends grep's capabilities beyond basic regular expressions.

Lets take an example Wordle to get a feel for how the process works. Of course we need a starter word - I'm using **TALES** which [analysis](https://www.sfi.ie/research-news/news/wordle-data-analytics/) has found to be a successful starter word. I've chosen Wordle #801 at random as an example from the [Wordlearchive](https://wordlearchive.com/801) website.

![First Wordle Guess](\assets\img\Pasted_image_20241207175648.png)

After our first guess we have two green letters - how do we incorporate this into our grep search? We can use our positive lookahead to check for words that have A as the second letter and E as the fourth letter. We do this as follows:

```bash
grep -Pi "(?=^.a.e.)" "Wordle Words.txt"
```

The caret symbol `^` asserts that the following pattern occurs at the start of a line (so that we don't get matches in the middle of words). The `.` symbol represents any character except for line terminators so we are basically saying that we don't care about the other letters in the word so long as the second letter is an A and the fourth letter is an E.

Already this search has whittled down our choice of words from over 3,000 to around 150, however we can do better than this as we still have more information we can use. In addition to knowing the second and fourth letters, we also know what letters are **not** in the word - i.e. we know the word does **not** contain T, L or S.

We can incorporate this information with by piping our first result to an additional **inverse** grep search using the `-V` flag. In our inverse search we can use the logical **OR** operator `|` to specify that we know the word doesn't contain the letters T, L or S. Our command now looks like this:

```bash
grep -Pi "(?=^.a.e.)" "Wordle Words.txt" | grep -Piv "(?=.*t|l|s)"
```

We should now be down to around 84 candidates - pretty good after the first guess! Our word list should be formatted alphabetically which helps with sense-checking the output. If we see one of the disallowed letters in the result, we know something has gone wrong with the logic of the search or flags.

At this point we need to make a guess for our next word. I would like to introduce a Python tool called [wordgradient](https://github.com/ctosullivan/WordGradient) I developed especially for this purpose. Wordgradient sorts input words by language frequency, a factor that the creators of the WordleBot have acknowledged as being a consideration for solutions. If we pipe the output of our search to wordgradient we will be able to see which of our results is most commonly used in the English language according to the tool's dataset.

```bash
grep -Pi "(?=^.a.e.)" "Wordle Words.txt" | grep -Piv "(?=.*t|l|s)" | wordgradient -hd
```

![First Wordle Guess - Wordgradient](\assets\img\Pasted image 20241207175316.png)

Wordgradient has a head option `-hd` which I have used to display the top ten results only.

Lets go with **NAMED** for the next guess which is the third-most common word. I have chosen **NAMED** as the letters are unique and the second-most common result **NAKED** seems a bit odd for a second guess.

![Second Wordle Guess](\assets\img\Pasted image 20241207175623.png)

Ok no new letters turned green, but we do have some additional information to use in our search. Lets add the additional letters we know are not in the word to our query:

```bash
grep -Pi "(?=^.a.e.)" "Wordle Words.txt" | grep -Piv "(?=.*t|l|s|n|m|d)" | wordgradient -hd
```

![Second Wordle Guess - Grep](\assets\img\Pasted image 20241207180037.png)

Now we are down to 23 candidates with the most commonly used result being **PAPER**. Lets go with **PAPER** for our third guess.

![Third Wordle Guess](\assets\img\Pasted image 20241207180145.png)

Lots of green! We can probably guess the word at this point but lets run the search again to verify what results are remaining.

```bash
grep -Pi "(?=^.aper)" "Wordle Words.txt" | grep -Piv "(?=.*t|l|s|n|m|d)|(?=^p)"
```

We just need to update our search to include the letters we are sure of and update our inverse search to reflect the fact that we know the first letter is **not** p. Note that when we are checking a logical **AND** condition in the inverted search we need to use the `|` **OR** operator, as without this the search will end if the first condition isn't met. In the regular search there is no equivalent of a logical **AND** operator, however grep will keep searching while the assertions are being met which has the same effect.

Lets see what we are left with.

![Fourth Wordle Guess](\assets\img\Pasted image 20241207181014.png)

**CAPER!** There we have it - Wordle 801 solved in 4 guesses using regex!

## Conclusion

Using grep to solve Wordle is an enjoyable way to practice regex while tackling a fun puzzle. Each game helps you build confidence with regular expressions and logical thinking. So why not give it a try and boost your regex skills one Wordle at a time?

Happy coding—and puzzling!
