# Change your workflow directory with BASE

Drake by default assumes you want the working directory to be the same directory as your Drakefile.

When Drake starts, it assigns the working directory to a special variable, `BASE`. As Drake runs it looks for inputs in `BASE` and writes outputs to `BASE`.

A benefit of this is that you can run your Drake workflow from anywhere, and Drake will make sure to do all work in the same place as your Drakefile. This is a nice way to centralize your work.

However, you may not always want this behaviour. You can redefine the value of `BASE` if you wish Drake to use a different working directory while your workflow is running.

## Simple BASE example

```
; our base data directory
BASE=/backup/CPG

; Merge data from all sources combined.csv <­ amazon.csv, walmart.csv
  cat $[INPUTS] > $OUTPUT

; Filter out some bad stuff filtered.csv <­ combined.csv
  grep ­v “Scott’s Cakes” $INPUT > $OUTPUT

; Generate clusters file clusters <­ filtered.csv
  java ­jar resolve.jar $INPUT $OUTPUT
```

In the above workflow, all inputs will be read from the `/backup/CPG` directory and all outputs will be written to the `/backup/CPG` directory.

## Setting BASE on the command line

If $[BASE] is defined using := syntax (see variable redefinition), or not defined in the workflow file at all, you can set its value from the command­line, either by creating an environment variable BASE or using the `­­base` command­line flag.

## Subtask example

**subtask.drake**:
```
BASE=$[BASE]/$[SUBTASK_NAME]

; Even though subtask.drake is called twice, this step will not be
; duplicated and not create a conflict as long as output target’s name
; is unique after prefixing it with the base directory
output <­ input
 sort $INPUT > $OUTPUT
```

**Drakefile**:
```
BASE=hdfs:/tmp/hadoop­aaron/my­workflow
SUBTASK_NAME=US %call subtask.drake
SUBTASK_NAME=France %call subtask.drake
```

## Off-BASE files

If a filename starts with `!`, the `$[BASE]` variable has no effect on it. The `!` symbol is not considered a part of the name. This is convenient when you have a couple of external files in the workflow, or if you’re working on a workflow shared by other developers while modifying or debugging just a part of it. You can use other variables or just fully specify the filename. For example:

**Drakefile**:
```
BASE=hdfs:/apps/extract/poi/US/
!$[TMP]uuid_retention <­ input_records
  ...
```

**command line**:
```bash
; just build UUID retention step in my own temporary directory
TMP=hdfs:/tmp/aaron d =uuid_retention
```

## Warning!

If you find yourself wrestling with BASE throughout your workflow, you may be doing it wrong.

For example, you might redefine BASE in order to "trick" your workflow into exporting or sharing output files to somewhere else. A better approach might be to do all your work in the workflow directly, and then have specific steps to share or export certain outputs. Those scripts could read from configuration files to know what should be shared where. See **TODO!** for more on how to do this.

BASE is best used when you truly want your workflow, or some significant portion of your workflow, to operate in a specific directory other than where you have your Drakefile.
