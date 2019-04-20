---
id: 279
title: String Representation in Matlab is a Mess
date: 2019-04-20T16:59:00-05:00
author: apjanke
layout: post
categories:
  - Matlab
  - Computers
---

For historical reasons, string representation in Matlab data structures is something of a mess. The situation is looking up now, with the introduction of the `string` class and its double-quoted string literals in R2017a, but it's still messy because of back-compatibility baggage. This post lays out the situation as I see it. The purpose of this post isn't to complain or rag on MathWorks; it’s to share what I understand of the sitation with other people so they can be more effective Matlab programmers, and maybe better programming language designers.

I sympathize with the MathWorks engineers’ situation here: the new `string` class is the right way to do strings. But back-compatibility considerations tie their hands considerably.

## Terminology

* _cellstr_ – A cell array whose elements contain `char` row vectors or `char` empties. This is a standard Matlab term.
* _charvec_ – A `char` array that is either a row vector (1-by-n) or a 0-by-0 empty. This is Matlab's standard way of representing a single string. “charvec” is a word I made up myself; you won‘t find it in other documentation.
* _mxArray_ – The Matlab interpreter’s internal data structure for representing a Matlab array. This is a term from Matlab’s External Interfaces API.

## Matlab string types

Matlab stores string data in at least three ways:

* A single string represented as a `char` vector
* A cellstr array
* Multiple strings represented as a 2-D `char` array
* A `string` object array
* Custom user-defined object arrays

Why so many ways? Because old versions of Matlab didn’t have the `string` class (or classes at all for that matter). 2-D `char` arrays and cellstrs each have their own problems, and `string` was added later to address that. But because Matlab prizes backwards compatibility, it can’t just make a clean break and do away with the old forms.

The `char` datatype that represents individual characters is a 16-bit unsigned integer that is interpreted as a Unicode code point. Essentially, it is UCS-2. Some people call it UTF-16, but that’s not quite right, because it doesn’t have support for surrogate pairs, which is what distinguishes UTF-16 from UCS-2. (Regardless, Matlab doesn’t seem to molest unrecognized code points, so you can pass UTF-16 data through Matlab `char`s and it seems to work out.) If you think of it as UTF-16, you won’t be far wrong.

A Matlab `char` is basically the same thing as a Java `char`.

A Matlab `char` is not the same as a C `char`! A C `char` is an 8-bit integer whose interpretation is determined by context. Essentially, it’s just a byte.

## “String” is ambiguous

When someone uses the word “string” in the context of Matlab programming, they might mean any of:

* The generic notion of string or textual data
* A `char` vector or array
* A cellstr
* A `string` object
* All of the above at the same time (in the case of functions that will accept any of the above as inputs)
* A `char` or cellstr, but not a `string` object (in the case of older functions that aren’t updated to work with `string`s yet)

You can’t tell which they mean without more context.

This is complicated by the fact that an old version of Matlab’s `isstring()` function used to return true for `char` and `cellstr`, but not for `string`s (which didn’t exist back then). And the new `isstring(x)` function doesn’t give the same answers as `isa(x, 'string')`.

In this post, when I say “string” I’m talking about the generic notion of string data, and when I say “`string`” (in fixed-width font) I mean specifically the `string` class or object.

## What’s wrong with charvecs and cellstrs?

Up until `string` came along, the main way of storing string data in Matlab was to store single strings as `char` row vectors, and arrays of strings as cellstrs.

The main problem with charvecs and cellstrs is that they don't fit in with Matlab’s main programming paradigm, which is that everything is an array, and scalars are just 1-by-1 arrays of the same type. Charvecs and cellstrs don't work like that. A single string is represented as a charvec, but it’s not scalar: `numel()` doesn’t return 1 for it. Conversely, a cellstr that represents _N_ strings is _N_ long as returned by numel. But you can’t use a scalar cellstr as a single string: most functions that want a single string as input won’t accept a scalar cellstr; you have to pop out the charvec that it contains.

And they don’t support the relational operations that most Matlab types do. `==`, `<`, and `>` don’t work at all for cellstrs. And for charvecs, they operate on the individual `char` elements instead of the entire string. So you can’t compare chars or cellstrs using `==` and friends; instead you have to use the special `strcmp`. And you can’t concatenate single charvec strings with `[...]` to produce an array of strings. This means you can’t use polymorphic code to write functions that could accept either strings or numbers; you have to write special-case code for string inputs.

A cellstr is not a distinct Matlab type: it is actually a `cell` that contains charvecs in each of its elements. This means there’s nothing to prevent you from assigning or concatenating something besides a charvec into one of the cells, silently making the array no longer a valid cellstr. And to test whether an array is a cellstr, like `iscellstr()` does, it can’t just look at the type of the array; it has to go through and check the type and size of each cell’s contents. That introduces some overhead.

Charvecs also have a weird discontinuity at the empty string case: In general, a charvec represents an N-character-long string as a 1-by-N `char` array. Except for empty strings, which are usually represented by a 0-by-0 array instead of a 1-by-0 array.

## What’s wrong with 2-D char arrays?

A list of N strings is represented as an N-by-M `char` array, where M is the length of the longest string (or greater). This means that none of the Matlab indexing operations act as normal: `c(i)` is not the i-th string; lists of strings as 2-D `char` can’t generally be concatenated with the normal `[...]` operations; the results of `strlength()` and `ismember()` aren’t the same size as the input; `numel()` won’t give you the number of strings; `length()` – well, just don’t use `length()`. `==` doesn’t work like you’d want. No polymorphic coding here.

Matlab arrays are arranged in memory in [column-major order](https://en.wikipedia.org/wiki/Row-_and_column-major_order). But strings in 2-D `char` arrays are read in row-major order. This means that characters that are next to each other in a string _are not next to each other in memory_. To reassemble a single string, Matlab has to “stride” through memory to grab its constituent bytes. The more strings in your 2-D `char` array, the further apart the characters of an individual string will be. For large arrays, this could be bad for cache behavior.

## Chars and cellstrs in memory

A cellstr array is composed of one big array at the cell level, and then one array for each of the charvecs stored inside the cells. This means that a 1,000-long cellstr array has 1,001 mxArrays. These arrays are heap-allocated and could be stored anywhere throughout memory. Additionally, because there are lots of arrays, large cellstr arrays (especially those containing long strings) can cause memory fragmentation, even after they are cleared by Matlab.

A 2-D `char` array is a single mxArray holding char data that is stored in a single contiguous block of memory. This makes it more compact when at rest (aside from the striding issue mentioned above), though it does mean you’ll need to find larger contiguous blocks of memory to store it in.

However, cellstrs have the nice property that they can share their underlying `char` data for strings that have the same value. This means they can be very efficient for large arrays of low-cardinality data (that is, arrays that have a low number of distinct values compared to the total number of elements). This value-sharing doesn’t happen automatically — Matlab won’t automatically de-duplicate your strings for you — but happens whenever you make assignments from one cellstr element to multiple locations. For example:

```
c = repmat('X', [1 512]);
cstr = cell([1 1024*1024]);
cstr(:) = {c};
cstr2 = repmat({c}, [1 1024 * 1024]);
```

Each of `cstr` and `cstr2` is only using 1 KB for its underlying `char` data instead of 1 GB. Don’t believe me? Try `cstr3 = repmat(cstr2, [1 1024])` and see if it actually takes up a terabyte of RAM.

And don’t believe what `whos` is telling you about their memory usage. It’s double-counting:

```
>> whos
  Name       Size                            Bytes  Class    Attributes

  c          1x512                            1024  char               
  cstr       1x1048576                  1191182336  cell               
  cstr2      1x1048576                  1191182336  cell               
  cstr3      1x1073741824            1219770712064  cell               
```

(BTW, notice how long that `whos` call takes? That’s because of how N-long cellstrs are stored as N+1 mxArrays; Matlab has to crawl through memory and check each of them for their size.)

You can use a "uniquification" trick to de-duplicate the underlying `char` data in your cellstrs. This is nice to use on possibly-low-cardinality column data coming back from a SQL database query, or a CSV file containing addresses with a lot of repeated state and country names, for example.

```
function out = uniquify_trick(strs)
  [uStrs,ix,jx] = unique(strs);
  out = uStrs(jx);
end
```

After running through that function, you’ll have a cellstr array with the same values, but all the elements that have the same value will be sharing the same underlying `char` data.

## Strings and MAT files

In memory, cellstrs are generally preferable to 2-D chars. But when reading and writing MAT files, the efficiency of cellstrs is _terrible_. This applies to both read/write speed and size of the resulting MAT files.

I think this is due to the per-array overhead of having each single string in its own separate mxArray. Matlab has to write the per-array structures and do compression on each of these arrays individually.

2-D `char` arrays perform much better with MAT files. A 2-D `char` array is a single mxArray with its underlying data contiguous in memory, so it writes fast and often compresses well.

If you’re saving large string data sets to MAT files, do yourself a favor and convert them to 2-D `char`s before writing, and then convert back to cellstrs when you read them back.

## How do `string` objects work?

A `string` array behaves like a normal Matlab array with respect to the main array operations: `()`-indexing works as normal; concatenation is uniform; `str(1)` gets you a scalar string that is a 1-by-1 array of the same type as the input and can actually be used for stuff. `==`, `<`, and `>` all work as you’d want. It’s type-safe, insofar as you can’t assign non-strings into it, and `class(x)` tells you everything you need to know about its type. This is good.

The `string` class defines an interface but doesn’t directly expose its internal storage type. It might be just a thin, strongly-typed object wrapper around what is effectively a cellstr. I suspect it is. But it doesn’t have to be! The underlying data might actually be stored as a single array of `char`s with indexes into it, or it could even be packed UTF-8 data or something. And because this isn’t exposed, MathWorks could change the internal storage structure in future Matlab releases if there’s a reason to.

How do `string`s work with MAT files? I don’t know! I haven’t experimented with it yet. That’s something to do.

## Java strings

Java `char` arrays will automatically convert to Matlab `char` arrays. But Java `String` objects or arrays will not automatically convert to Matlab `char` or `cellstrs`. Why not? Because Java objects have _identity_ in addition to value, and that identity would be lost if Matlab did automatic conversion on them.

Converting Java objects to Matlab values is expensive. If you want to go fast passing large string arrays between Matlab and Java, convert them to packed `char` arrays and index vectors before handing them across the Matlab/Java boundary, and then convert to whatever you want back on the other side.

## So what should I do?

I haven’t programmed with `string` enough to be sure, but I think the right thing to do is: If you’re working with a recent version of Matlab, just use `string`s everywhere you can, and fall back to charvecs or cellstrs where you have to. Whenever you see doco or a Stack Overflow answer that says to use cellstrs, just read that as `string`s instead.

## The Octave situation

For those of you programming GNU Octave, the situation is a little worse. Octave does not have a `string` class. So you can’t use it. And it has its own syntax for double-quoted string literals: they produce `char` vectors instead of `string` objects. Which is fine, except it presents a compatibility issue for Matlab code, and prevents Octave from implementing its own `string` type that uses that literal syntax. Oh well.

Octave `char`s are not the same as Matlab `char`s. They are 8-bit ints instead of 16-bit ints. In the past, the encoding situation was a little fuzzy, but I think they’ve settled on Octave `char`s being UTF-8 Unicode data, in most situations.

