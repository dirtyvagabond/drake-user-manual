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

A Drake workflow starts with a simple text file that specifies the steps in your worklfow along with their inputs and outputs. As an example let's build and run a simple Drake workflow that takes a text file as it's input and outputs word frequencies. Create a file named `Drakefile` and put this in it[1]:

```
word-freqs.txt <- book.txt
  tr -cs A-Za-z '\n' < $INPUT |
  tr A-Z a-z |
  sort |
  uniq -c |
  sort -rn |
  head -n12 > $OUTPUT
```

You now have a Drake workflow with exactly one step. The input and output of the step is specified on the first line: we're expecting `book.txt` as the single input into the step, and we're expecting the step to produce `word-freqs.txt` as the single output. 

The other lines in the step carry out the actual work. In this case it's code written for bash, which is Drake's default step language, or "protocol". The bash code processes the input file and pipes it through a handful of pre-existing bash tools and writes the final results to the output file. When writing steps as bash we can refer to our input as `$INPUT` and our output as `$OUTPUT` because Drake prepares bash's environment with these and other values before shelling out to bash. The variable substitution helps keep workflows flexible and easy to manage. If you later choose to change the name of the input from `book.txt` to `tome.txt`, for example, you need only change it in the first line of the step.

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

Drake is telling you it's going to run one step, and explains why: the output file `word-freqs.txt` doesn't exist yet, and this one step will create it for us. Enter 'y' to confirm. Drake will run your workflow and you should see output like this:

```
Running 1 steps with concurrence of 1...

--- 0. Running (missing output): /Users/aaroncrow/workspace/drake/demos/McIlroy/././word-freqs.txt <- /Users/aaroncrow/workspace/drake/demos/McIlroy/././book.txt
--- 0: /Users/aaroncrow/workspace/drake/demos/McIlroy/././word-freqs.txt <- /Users/aaroncrow/workspace/drake/demos/McIlroy/././book.txt -> done in 0.03s
Done (1 steps run).
```

Drake completed the workflow, producing the word frequences file that we wanted. Open up `word-freqs.txt` to see the specific results generated from your `book.txt` input.

If we try to run the workflow again, Drake tells us it's not necessary:

```
drake

Nothing to do.
```

This is because Drake sees that the input has not changed since the output was generated. If you want to force the issue, you can tell Drake to rerun anyway:

```
drake +...
```

Drake lets us control which outputs (a.k.a., "target selection") in our workflow to force build. We use `+...` to mean "everything", which in this case forces a run of the one and only step in our workflow. We will cover target selction in finer detail later.

Another way to see this workflow run again is to `touch` the input file, then tell Drake to run normally...

```
touch book.txt && drake
```

Drake will choose to rerun the step and re-write `word-freqs.txt` because it sees that the input is now newer than the existing output.

## Human Resources Workflow

Let's look at a "Human Resources" workflow that demonstrates Drake's support for multiple protocols, multiple inputs, and multiple outputs. The full workflow is available in Drake's public repo, under the `demos` directory:
https://github.com/Factual/drake/blob/develop/demos/human-resources/Drakefile

Here's what we see when we graph it:
```bash
drake --graph
open drake.png
```

![alt text](https://github.com/dirtyvagabond/drake-user-manual/blob/master/images/human-resources-workflow.png "Human Resources Workflow, graphed")

You can see that one of the steps takes multiple inputs: `skills` and `people` are used to produce the `skills.people` output. Also note that one of the steps produces multiple outputs: `people.json` is an input to a step that produces `last_gt_first.txt` and `first_gt_last.txt`. Drake provides a lot of flexibility here; a Drake step can require as many inputs as you like and produce as many outputs as you like. You can also reuse inputs and outputs throughout your workflow.

Let's examine each Human Resources step individually...

```
;
; Filters everyone down to just the people with the right numbers.
;
; Output is like:
;   Artem Boytsov,310-100-0000
;   Vinnie Pepi,+86-310-400-0000
;   Will Lao,+86-310-600-0000
;
people <- everyone
  grep 310 $INPUT | sort > $OUTPUT
```

```
;
; Joins people to their skills. Output is like:
;   Aaron Crow,java ruby clojure, 310-300-0000
;   Artem Boytsov,flying southwest, 310-100-0000
;   Maverick Lou,java clojure jenkins, +86-310-200-0000
;
; Includes a bit of debug output.
;
people.skills <- skills, people
  echo Number of inputs: $INPUTN
  echo All inputs: $INPUTS
  echo Joining ...
  join -t, $INPUTS > $OUTPUT
```

```
;
; Adds UUIDs and formalize to JSON.
; Uses Drake's python protocol, for inline Python.
;
people.json <- people.skills [python]
  import csv
  import json
  import uuid
  outfile = open('$[OUTPUT]', 'w')
  with open('$[INPUT]', 'rb') as csvfile:
    for row in csv.reader(csvfile):
      jsn = {'name': row[0], 'skills': row[1], 'tel': row[2]}
      jsn['uuid'] = str(uuid.uuid1())
      outfile.write("{0}\n".format(json.dumps(jsn)))
```

```
;
; Generates 2 reports:
;   1) All people whose last name is longer than their first
;   2) All people whose first name is longer than their last
;
; Uses Drake's ruby protocol, for inline Ruby.
;
last_gt_first.txt, first_gt_last.txt <- people.json [ruby]
  require 'json'
  lGf = File.open("$[OUTPUT0]", "w")
  fGl = File.open("$[OUTPUT1]", "w")
  File.open("$[INPUT]").each do |line|
    rec = JSON.parse(line)
    first, last = rec['name'].split(" ")
    lGf.puts(rec['name']) if last.length > first.length
    fGl.puts(rec['name']) if first.length > last.length
  end
```

```
;
; No JSON files outside of Engineering; only CSV is acceptable!
; Let's translate to CSV and suggest usernames while we're at it.
;
; This uses Drake's experimental Clojure-based c4 protocol, which
; knows how to automagically travel between TSV, CSV, JSON formats.
;
for_HR.csv <- people.json [c4row]
  (let [[fname lname] (str/split (row "name") #" ")]
    (assoc row :uname (str (first fname) lname)))
```

## Artem's "People Skills" Workflow

If you'd like a screencast companion to walk you through a fun Drake workflow, check out Artem's [introductory screencast for Drake](http://www.youtube.com/watch?v=BUgxmvpuKAs)[2]. The workflow he demonstrates is available in Drake's public repo, under the `demos` directory:
https://github.com/Factual/drake/tree/develop/demos/people-skills

## Footnotes

1. Thanks Dr Drang for [the story about Doug McIlroy](http://www.leancrew.com/all-this/2011/12/more-shell-less-egg)
2. We're tickled by the relative popularity of Artem's video, which has been viewed over 7,000 times. Or as Artem once put it: "Since launch, people spent cumulative 639.8 hours watching Drake tutorial on Youtube, which is not Apache Hadoop, of course, but still pretty neat. :)". http://www.youtube.com/watch?v=BUgxmvpuKAs
