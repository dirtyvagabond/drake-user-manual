# Why Drake

## Elevator Pitch and Some History

Processing data can be a real a mess! Here are just a few of the issues:

* A multitude of steps, with complicated dependencies
* Code and input can change frequently – it’s tiring and error-prone to figure out what needs to be re-built
* Inputs scattered all over (home directories, NFS, HDFS, etc.), tough to maintain, tough to sustain repeatability

We read with interest an article called "[Make for Data Scientists](http://blog.kaggle.com/2012/10/15/make-for-data-scientists/)"[1] by Paul Butler, a self-described Data Hacker. The article explored the challenges of managing data processing work. Paul went on to explain why GNU Make could be a viable tool for easing this pain. He also pointed out some limitations with Make, for example the assumption that all data is local.

We were gladdened to read Paul’s article because we’d been hard at work building an internal tool to help manage our data workflows. A defining goal was to end up with a kind of “Make for data”, but targeted squarely at the problems of managing data workflow. We named this tool Drake and [released it as open source](http://blog.factual.com/introducing-drake-a-kind-of-make-for-data) on January 24th, 2013[2]. We were delighted when the project gained some amount of attention, including [an epic knock-down-drag-out set of threads on Hacker News](https://news.ycombinator.com/item?id=5110921)[3]. This helped inspire us to continue with the project and mature Drake into a valuable data processing workflow tool.

## Some Benefits of Drake

Drake is a text-based command line data workflow tool that organizes command execution around data and its dependencies. Data processing steps are defined along with their inputs and outputs.  It automatically resolves dependencies and provides a rich set of options for controlling the workflow. It supports multiple inputs and outputs and has HDFS support built-in.

We use Drake at Factual on various internal projects. It serves as a primary way to define, run, and manage data workflow. Some core benefits we’ve seen:

* Non-programmers can run Drake and fully manage a workflow
* Encourages repeatability of the overall data building process
* Encourages consistent organization (e.g., where supporting scripts live, and how they’re run)
* Precise control over steps (for more effective testing, debugging, etc.)
* Unifies different tools in a single workflow (shell commands, Ruby, Python, Clojure, pushing data to production, etc.)

Drake offers a ton more stuff to help you bring sanity to your otherwise chaotic data workflow, including:

* rich target selection options
* support for inline Ruby, Python, R, and Clojure code
* tags
* ability to “branch” your input and output files
* HDFS integration
* variables
* includes

Drake makes it refreshingly easy to spin up proof of concepts. We like to use Drake to express evolving data workflows that will be shared across a wide team. And Drake is especially useful for tying together heterogeneous scripts and command line calls, thereby taming an otherwise hodgepodge of tasks. We also like Drake for helping us manage chains of Hadoop jobs that relate to each other.

## Tradeoffs

Drake is not great if you just need to glue together some existing same-language code. That can usually be done more simply by staying within the borders of that language. Doubly so if you plan to share your work with others who don’t already know and understand Drake workflows.

Drake does not fully answer all problems in the data workflow space. A leading example: how to manage long running tasks? For tasks that take a long time and do a large amount of work, you often want features like resumability and trackability. At Factual, we have a custom built task management service called Vineyard that addresses these and other similar issues for us. We have glue that allows Vineyard and Drake to work together in various ways, but Drake out-of-the-box doesn’t offer these kinds of features for long running tasks.

## Footnotes

1. http://blog.kaggle.com/2012/10/15/make-for-data-scientists
2. http://blog.factual.com/introducing-drake-a-kind-of-make-for-data
3. https://news.ycombinator.com/item?id=5110921
