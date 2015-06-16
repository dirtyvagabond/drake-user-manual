# Getting Started with Drake

In this chapter we'll install Drake and then demo several simple workflows.

## Install Drake

Drake includes a Linux and Mac friendly install script.

1. Download the [drake script](https://github.com/Factual/drake/blob/develop/bin/drake)
2. Place it on your $PATH where your shell can find it (eg. `~/bin`)
3. Set it to be executable (eg. `chmod a+x ~/bin/drake`)
4. Run it (`drake`) and it will install Drake

Alternatively there's a Homebrew formula for Drake:
```
brew install drake
```

Once you've installed Drake you can verify by asking for the version:
```bash
drake --version
```

You should see something like:

```bash
Drake Version 0.2.0
```

If you are a Windows user wishing to manage data processing workflows... we are very sorry for that.


## Your First Workflow

Let's build and run a simple Drake workflow that takes a text file as it's input and outputs word frequencies. Create a file named `Drakefile` and put this in it[1]:

```
word-freqs.txt <- book.txt
  tr -cs A-Za-z '\n' < $INPUT |
  tr A-Z a-z |
  sort |
  uniq -c |
  sort -rn |
  head -n12 > $OUTPUT
```

Now create a file named `book.txt` and paste a bunch of text (any text, but preferably something meaty and of lofty literary quality). Make sure this new file is in the current directly (where you also have your `Drakefile`. Now you can run Drake on it:

```bash
drake
```

Drake gives you a brief summary of what it plans to do:

```bash
The following steps will be run, in order:
  1: /Users/aaroncrow/workspace/drake/demos/McIlroy/././word-freqs.txt <- /Users/aaroncrow/workspace/drake/demos/McIlroy/././book.txt [missing output]
Confirm? [y/n]
```

Drake is telling you it's going to run one step, and explains why: the output word-freqs.txt doesn't exist yet, and this one step will create it for us. Enter 'y' to confirm. Drake will run your workflow and you should see output like this:

```
Running 1 steps with concurrence of 1...

--- 0. Running (missing output): /Users/aaroncrow/workspace/drake/demos/McIlroy/././word-freqs.txt <- /Users/aaroncrow/workspace/drake/demos/McIlroy/././book.txt
--- 0: /Users/aaroncrow/workspace/drake/demos/McIlroy/././word-freqs.txt <- /Users/aaroncrow/workspace/drake/demos/McIlroy/././book.txt -> done in 0.03s
Done (1 steps run).
```

Drake completed the workflow, producing the word frequences file that we wanted. Open up `word-freqs.txt` to see the specific results, generated from your `book.txt` input.

## Artem's "People Skills" Workflow

Here is a 4 step workflow that filters and sorts people skills for us. This is the workflow that Artem demonstrates in his [introductory screencast for Drake](http://www.youtube.com/watch?v=BUgxmvpuKAs)[2].

```
;
; Cleans up the skills file.
; Input lines are like:
;   Artem Boytsov,flying southwest
;   REAL BAD ENTRY
;   Maverick Lou,java clojure jenkins
;
skills.filtered <- skills
  grep -v BAD $INPUT > $OUTPUT

;
; Starts with people formatted like...
;   Crow,Aaron,310-300-0000
;   Pepi,Vinnie,+86-310-400-0000
;   Lao,Will,+86-310-600-0000
;
; ...and ends with people.fullname formatted like:
;   Aaron Crow, 310-300-0000
;   Vinnie Pepi, +86-310-400-0000
;   Will Lao, +86-310-600-0000
;
people.fullname <- people
  awk -F, '{ print $2 " " $1 ", " $3}' $INPUT > $OUTPUT

;
; Simple sort of our people...
;
people.sorted <- people.fullname
  sort $INPUT > $OUTPUT

;
; Joins sorted people to their skills. output is like:
;   Aaron Crow,java ruby clojure, 310-300-0000
;   Artem Boytsov,flying southwest, 310-100-0000
;   Maverick Lou,java clojure jenkins, +86-310-200-0000
;
output <- skills.filtered, people.sorted
  echo Number of inputs: $INPUTN
  echo All inputs: $INPUTS
  echo Joining ...
  join -t, $INPUTS > $OUTPUT
```

## Footnotes

1. Long ago I did some clumsy Googling to find some nice bash-fu for doing something nifty with data. Thanks Dr Drang for [the story about Doug McIlroy[http://www.leancrew.com/all-this/2011/12/more-shell-less-egg]
2. We've been tickled by the relative popularity of Artem's video, which has been viewed over 5,000 times. Or as Artem once put it: “Since launch, people spent cumulative 639.8 hours watching Drake tutorial on Youtube, which is not Apache Hadoop, of course, but still pretty neat. :)”. http://www.youtube.com/watch?v=BUgxmvpuKAs
