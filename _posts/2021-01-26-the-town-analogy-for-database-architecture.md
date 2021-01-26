---
title: "The Town Analogy for database architecture"
date: 2021-01-26T07:24:00
author: apjanke
layout: post
categories:
  - Computers
---

The other day, I was talking with a colleague about the difference between table scans, index scans, and index seeks, and I came up with this analogy to explain how they work in SQL Server.

---------------------------------------------------------------------------------------

A database table is like a town. The people in the town are your data. The houses they live in are disk blocks (or "pages"). The plot of land the town sits on is your disk.

You work in a little office up on a hill just past the outskirts of town. That's your computer's main memory (RAM).

The town's phone book is an index.

Let's say you want to find Mary Keating. You could go to town, start at one end, and knock on every single door in town and ask "Is Mary Keating here?". That's a table scan.

You could take the phone book, read the entire thing front to back, and make a note each time you find "Mary Keating", and then go to the address you found. That's an index scan.

Or, since you know the phone book is sorted alphabetically, you could open it up in the middle, flip right to the Ks, then flip to the KEs, then to the KEAs, and pretty soon you've found Mary, and you only had to look at a few pages out of the entire phone book. That's an index seek.

Whenever you knock on the door of a house, all its occupants come to the door at once, so you get to talk to all of them at the same time with a single knock. That's "block IO".

Let's say you want to talk to all of Mary Keating, Billy Keating, Sally Keating, Marvin Keating, and Susie Keating. You look them all up in the phone book and see that they all have the same address, so you get to talk to all of them with just one visit to a single house. They all live in the same house, because they're all Keatings. That's how "CLUSTERED" behavior works.

There's a diner next to your office at the top of the hill. Every time you go in to town to talk to someone, they and their whole household follow you back up the hill and have dinner at the diner. The townspeople are lazy, so they just stay and hang out at the diner until someone else shows up and wants their table. If you've talked to someone recently, and want to talk to them _or anyone else who lives in their house_ again, you can just zip over to the diner instead of going all the way in to town. This is the page cache, or just "cache".

Sometimes you get phone calls from people other towns, asking about the people in your town. Those are network requests.

One day someone from another town calls you up and wants to find out the birthday for "Mary Keating Go Screw Yourself". You look up Mary Keating's name in the phone book, call her up, and say, "Hi, Mary Keating Go Screw Yourself!" Mary takes offense and never talks to you again. You've lost your data. This is a "SQL injection attack".

Let's say you're a serial killer. You come to town, and you chop up all the inhabitants, and you put all the arms in one warehouse, and the legs in another warehouse, and the heads in another, and the torsos in another. Now, if one day you're interested in legs, you can just go to the Leg Warehouse and sort through all the legs, without having to shuffle through all the arms, heads, and torsos that you're not interested in that day. That's a "column store database".
