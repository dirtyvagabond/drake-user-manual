# Change your workflow directory with BASE

Drake by default assumes you want the working directory to be the same directory as your Drakefile. So Drake reads inputs from there and writes outputs to there, regardless of where you called Drake from.

Drake tracks the working directory in the special variable `BASE`. You can redefine the value of `BASE` if you wish Drake to use a different working directory while your workflow is running.

## What is BASE?

BASE is a special variable in Drake. Its value is automatically added to every input or output file name in the workflow. It defines the base directory for all data files. It can contain a file system prefix, and because of the behavior of variables described above, it can be redefined in nested files easily, providing a crude method for creating reusable procedures in Drake.

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

If $[BASE] is defined using := syntax (see variable redefinition), or not defined in the workflow file at all, you can set its value from the command­line, either by creating an environment variable BASE or using the `­­base command­line` flag.

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
