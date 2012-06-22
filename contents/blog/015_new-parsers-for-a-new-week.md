---
title: New Parsers for a New Week
time: 2012/06/21 10:00
tags: biopython, gsoc, python
---

It's that time of the week again, for another SearchIO update. This week I added two new format support: `hmmer-tab` and `hmmer-domtab`. These are formats produced by the HMMER suite; the first triggered by using the `--tblout` flag, and the second one by using `--domtblout` flag. For now, both formats are fully supported for all three SearchIO operations: parsing/reading, indexing (both in memory and with a database), and writing. In this post, I'd just like to explain a bit on these two formats as there are some 'gotcha's that users should be aware of.

The first one is about the `hmmer-domtab` parser. Officially, `hmmer-domtab` does not exist with that name. There is an issue with the output alignment coordinates in a domain table file which prevents the use of that name. Instead, users must specify the program they use as the format name. So for now, parsing domain table files works with one of these three names: `hmmscan-domtab`, `hmmsearch-domtab`, or `phmmer-domtab`. 

Why do we need to do this? Basically, it boils down to the way HMMER outputs its query and hit HSP coordinates. SearchIO tries to standardize all its HSP coordinates into `query_from`, `query_to`, `hit_from`, and then `hit_to`. Howeverm HMMER output files use a different convention: 'hmm from', 'hmm to', 'ali from', and 'ali to'. Depending on the search program used, 'hmm from' could either be mapped into `query_from` (e.g. in hmmsearch) or into `hit from` (e.g. in hmmscan). This was not an issue with the plain text format since there is information about the program, so the parser can do the necessary switch. This was also not an issue in `hmmer-tab`, since it does not output any coordinates. However, the domain table files output these coordinates without any information on the program used to produce it. So without specifying which program to use, there is no way for the parser to know, for example, if it needs to put 'hmm from' into `query_from` or `hit_from`.

The second one is about using them for conversion. Unlike BLAST, where we can use `SearchIO.convert` for converting `blast-xml` into `blast-tab`, users can not use `SearchIO.convert` for `hmmer-text`, `hmmer-tab`, or `hmmer-domtab`. This is because in each of these three formats, there is information not output by the others that the parser can not compute. For example, in `hmmer-tab`, one of the columns (labelled `clu`) contains information about how many domain clusters are present in the hit. This is not present in the other two formats. Another example is the `tlen` column in the domain table files, which denotes how long the full Hit sequence is. For this one, users can try to supply the information themselves (e.g. by pulling it from another source), but the parser will not attempt any guess on its value.

That's all for this week. As always, if you are curious to try the new branch, it is available [here](https://github.com/bow/biopython/tree/searchio). For the coming weeks, since the core SearchIO API is now complete (`read`, `parse`, `index`, `index_db`, `write`, and `convert`), I will focus on adding new formats to support and improving the code + documentation.