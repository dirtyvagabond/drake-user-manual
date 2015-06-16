# Getting Started with Drake

In this chapter we'll install Drake and demo a few simple data processing workflows.

## Install Drake

As of version 0.2.0, Drake ships with a Linux and Mac friendly bash install script that will automagically install Drake for you.

1. Download the [drake script](https://github.com/Factual/drake/blob/develop/bin/drake)
2. Place it on your $PATH where your shell can find it (eg. `~/bin`)
3. Set it to be executable (eg. `chmod a+x ~/bin/drake`)
4. Run it (`drake`) and it will install Drake

Alternatively there's a Homebrew formula for Drake and we do our honest best to keep it up to date:
```
brew install drake
```

The above should work fine with most Linux environments. Once you've installed Drake, verify by asking for the version:
```bash
drake --version
```bash

You should see somethign like:

```bash
Drake Version 0.2.0
```

If you are a Windows user wishing to manage data processing workflows, we are very sorry for that.


## A Simple Data Processing Workflow


