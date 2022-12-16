---
feature: phil-wadler
authors:
  - "ice1000"
start_date: "2022/12/01"
---

# Wadler-style pretty printing API for SQL

This RFC proposes a new API for pretty-printing pseudo "structures".
The purpose of the API is to supersede the current implementation of SQL explain.

## Goals

+ Make the output of SQL explain "cooler" in the sense that ASCII (or Unicode) art
  are used to help with the readability of the output.
+ Implementation-wise, the API should be extensible and not tightly coupled with
  the actual SQL syntax so that it can be used for other purposes.
+ The current implementation is going to be replaced by the new API, if things went well.

## Non-goals

+ The new API is not designed for performance-critical applications.
  SQL explain is not considered to be such an application.
+ The new API does not aim to be super flexible like the `pretty` crate.
  This gives us spaces to simplify the design and implementation.

## Intended behavior

+ Users specify a preferred width, usually the width of the terminal,
  or 80 or 120, etc.
+ The API automatically calculates the actual width of the output,
  based on the preferred width.
  + If everything can be done in one line, the actual width is the line's width, and the output will be one-linear.
  + If the output cannot be done in one line, the output will try to break down the output into multiple lines, and retry to fit the output into the preferred width for every line.
+ The API supports wrapping the output with beautiful ASCII art.

## Types

### Enum `Pretty` for pretty printing

+ It represents an object that can be displayed as a string.
+ The width and height of the pretty-printed string can be calculated in advance.
+ Objects that implement `Pretty` are hereafter called "pretty" or "pretties".

Variants:

+ Variant `Record` that brutally pretty-prints a struct-like data.
  + It contains a list of name-pretty pairs.
+ Variant `Array` that brutally pretty-prints an array-like data.
  + It contains a list of pretties.
+ Variant `Text` that pretty-prints a string.
  + It contains a copy-on-write string.

### Record `PrettyConfig` for pretty printing configuration

It contains indentation, preferred width, etc.

## Important methods

+ `Pretty::ol_len(&self) -> usize`
  + Returns the length of the pretty-printed string, under a one-linear setting.
