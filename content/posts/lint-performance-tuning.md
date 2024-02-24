+++ 
draft = false
date = 2024-02-24T15:21:45+01:00
title = "Lint Performance Tuning"
description = ""
slug = ""
authors = ["Tomas Havlicek"]
tags = [
    "android",
    "lint"
]
categories = []
externalLink = ""
series = []
+++

A few days ago, a member of the development team raised an issue regarding the prolonged duration
of the quality stage, which was averaging 16 minutes. In comparison, both build and test stages which
run in parallel before the quality stage runs for approximately 13 minutes.

## Initial Assessment

The tasks executed for each stage are as follows:

- Build: `assembleRelease`
- Test: `koverVerifyRelease`
- Quality: `lint`

At first glance, nothing out of order. 

The 8 GB of memory also seemed like enough.

[![Develocity Performance](/images/initial-assesment.png)](/images/initial-assesment.png)

## Lint Configuration

The `ignoreTestSources` flag was already set to `true`. Because of that lint ignores all test sources.
The `checkAllWarnings` flag, which enables all warnings, was disabled. The project contains two modules,
sample app and a library. Since sample app only purpose is to test the library integration, 
there's no gained benefit to include in lint analysis. Therefore I updated the quality stage task 
to `library:lint`. This change didn't produce any time save, because `sampleapp:lint` task is not 
on the critical path.

[![Develocity Timeline](/images/lint-configuration.png)](/images/lint-configuration.png)

## Root Cause 

Since lint version 7, `lint` is just an alias for `lintDebug`. And `lintDebug` depends on `compileDebugKotlin` 
and by extend other Debug variant tasks, which were not cached from previous stages.
I was able to save on average 10 minutes on every quality stage run by changing `lint` to `library:lintRelease`.
In this project the quality stage runs around 700 times a month, resulting in a cumulative time 
saving of 116 hours of developers time. 

## Conclusion
Don't mix debug and release variant tasks in your CI/CD pipelines. 

[performance-tuning](https://googlesamples.github.io/android-custom-lint-rules/usage/performance-tuning.md.html)

