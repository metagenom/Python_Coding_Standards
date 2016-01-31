Python_Coding_Standards
=======================
A living specification documenting Python coding standards at Amplytica
-----------------------------------------------------------------------

#### The Zen of Python

    Beautiful is better than ugly.
    Explicit is better than implicit.
    Simple is better than complex.
    Complex is better than complicated.
    Flat is better than nested.
    Sparse is better than dense.
    Readability counts.
    Special cases aren't special enough to break the rules.
    Although practicality beats purity.
    Errors should never pass silently.
    Unless explicitly silenced.
    In the face of ambiguity, refuse the temptation to guess.
    There should be one-- and preferably only one --obvious way to do it.
    Although that way may not be obvious at first unless you're Dutch.
    Now is better than never.
    Although never is often better than *right* now.
    If the implementation is hard to explain, it's a bad idea.
    If the implementation is easy to explain, it may be a good idea.
    Namespaces are one honking great idea -- let's do more of those!
    
### Python 2 vs 3
All new software shall be written to support Python3 only as Python2 is now considered legacy. The Python3 baseline is version **3.4** for all new software. Legacy software may be maintained at its current version or ported to Python3 at management's discresion.

### Style Guides
In general follow [PEP8](https://www.python.org/dev/peps/pep-0008/) and the [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html) (which is superset of PEP8). **Read These!** **Follow These!**

#### Style Guide Exceptions
The following are practice we follow which are exceptions to the above style guides:

- **Shebang**
	- Shebangs should be in the format  ```#!/usr/bin/env python3``` not ```#!/usr/bin/python3``` as many system's CPython interpreter is found in a different location than ***/usr/bin/***  

### File Header Docstring
All files should contain a file header docstring directly below the shebang in the format below:


```
#!/usr/bin/env python

"""
Created by: Amplytica (YYYY)

Description: A program that takes query proteins and uses BLASTp to search for highly similar
             proteins within a local BLAST database derived from a target proteome. The program
             then uses BLASTp again to do a reverse search for found subject proteins in the query.
             It then filters hits down to Best Reciprocal Hits to confirm gene orthology.
             
Requirements: - This program requires the Biopython module: http://biopython.org/wiki/Download
              - This script requires BLAST+ 2.2.9 or later.
              - All operations are to be done with protein sequences.
              - All query proteins should be from sequenced genomes in order to facilitate reciprocal BLASTp.
              - MakeAABlastDB must be used to create BLASTn databases for both query and subject proteomes.
              - BLASTp requires that the FASTA file the subject database remain in the same directory as the database.
"""

# Imports:
import argparse

```

### Other Recommendations

#### Use functional programming paradigms
In a functional program, input flows through a set of functions. Each function operates on its input and produces some output. Functional style discourages functions with side effects that modify internal state or make other changes that aren’t visible in the function’s return value. Functions that have no side effects at all are called purely functional. Avoiding side effects means not using data structures that get updated as a program runs; every function’s output must only depend on its input.

Take a look at the [Functional Programming](https://docs.python.org/3.4/howto/functional.html) section of the Python documentation.
 

#### Store complex data structures as objects for improved readability
For example the code below involves a relatively complex filtering algorithm in which data is passed into the function via a list of lists where the outer list is a list of Hidden Markov Model hits and the inner list is a list of a particular hit's attributes. The code ```AlignmentOneLength = RowOne[-2] - RowOne[-3]``` requires the developer to remember what attribute is at what index inside the inner list which can lead to confusion and potential bugs.  

```
while i < (len(HMMHitTable) - 1):
		RowOne = HMMHitTable[i]  # Current Row in hit table.
		RowTwo = HMMHitTable[i + 1]  # Row below.
		if RowOne[0] == RowTwo[0]:  # If they have the same targe protein.
			AlignmentOneLength = RowOne[-2] - RowOne[-3]  # RowOne AliTo - AliFrom
			AlignmentTwoLength = RowTwo[-2] - RowTwo[-3]  # RowTwo AliTo - AliFrom
			Overlap = RowOne[-2] - RowTwo[-3]  # RowOne AliTo -  RowTwo AliFrom
			if Overlap > 0:  # If there is overlap...
				# If the overlap is greater than 50% of either alignment.
				if ((float(Overlap) / float(AlignmentOneLength)) > 0.5) or (
							(float(Overlap) / float(AlignmentTwoLength)) > 0.5):
					if RowOne[3] < RowTwo[
						3]:  # If row one has a lower E-value than row two remove row two else remove row one.
						HMMHitTable.remove(RowTwo)
					else:
						HMMHitTable.remove(RowOne)
					i -= 1  # Resets list index.
		i += 1
```

One can defining the parameters of a HMM hit as an object's properties.  

```
class HMMHit(object):
	def __init__(self, target_protein, hmm_name, score, e_value, hmm_from, hmm_to, ali_from, ali_to, hmm_length):
		self.target_protein = str(target_protein)
		self.hmm_name = str(hmm_name)
		self.score = float(score)
		self.e_value = float(e_value)
		self.hmm_from = int(hmm_from)
		self.hmm_to = int(hmm_to)
		self.hmm_overlap = self.hmm_to - self.hmm_from
		self.ali_from = int(ali_from)
		self.ali_to = int(ali_to)
		self.ali_length = self.ali_to - self.ali_from
		self.hmm_coverage = float(self.hmm_overlap) / float(hmm_length)
```

This allows us to have more explicit readable code. For example the code:

```overlap_between_hits = hit_one.ali_to - hit_two.ali_from``` 

is much more **explicit** and **concise** than:

```
AlignmentOneLength = RowOne[-2] - RowOne[-3]  # RowOne AliTo - AliFrom
AlignmentTwoLength = RowTwo[-2] - RowTwo[-3]  # RowTwo AliTo - AliFrom
Overlap = RowOne[-2] - RowTwo[-3]  # RowOne AliTo -  RowTwo AliFrom
```  

Using objects to store semistructured data make your code more readable and concise as seen below: 

```
while i < (len(hmm_hit_list) - 1):
		hit_one = hmm_hit_list[i]  # Current Row in hit table.
		hit_two = hmm_hit_list[i + 1]  # Row below.
		if hit_one.target_protein == hit_two.target_protein:
			overlap_between_hits = hit_one.ali_to - hit_two.ali_from
			if overlap_between_hits > 0:
				# If the overlap is greater than 50% of either alignment.
				if ((float(overlap_between_hits) / float(hit_one.ali_length)) > max_align_overlap) or (
							(float(overlap_between_hits) / float(hit_two.ali_length)) > max_align_overlap):

					if hit_one.e_value < hit_two.e_value:
						hmm_hit_list.remove(hit_two)
					else:
						hmm_hit_list.remove(hit_one)
					i -= 1  # Resets list index.
		i += 1
``` 
