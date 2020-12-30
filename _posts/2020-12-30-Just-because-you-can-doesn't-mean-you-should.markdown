---
layout: post
title:  "Just because you can, doesn't mean you should"
date:   2020-12-30 15:20:00 +0000
---
There are many things which are possible, but that doesn't mean you should do them for example `Dictionary<string, List(Tuple<Int32, string, Boolean>)>`

When deciding and using DataTypes it's important to pick the simplest DataType that matches your needs, however the example above, seems to overcomplicate things greatly. 

For starters I have never even heard of a tuple (which wouldn't be an issue) however I still don't have a clue what a dictionary of a string and list of tuples would even look like never mind how I would implement it in code, which is one of the main issues I have with it,that it is very difficult to visualise whats stored in it and how it looks.

For example `new List<String>() { "abc","def","egh" }` can be visualised as:

* abc
* def
* ghi

`New Dictionary(Of Integer, String) From { { 1, "Test1" }, { 2, "Test1" } }` can be visualised as:

| Key | Value |
| --- | --- |
| 1 | Test1 |
| 2 | Test2 |

Or

| 1 | 2 |
| --- | --- |
| Test1 | Test2 |