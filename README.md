# Introduction
This project implements the Burrows-Wheeler Transform (BWT) algorithm to manipulate and encode a given string with the purpose of data compression. Additional properties of the BWT are also leveraged here to create a string-matching tool that returns the position of the queried 'sub-string' within the larger string. 

# Pseudocode
```python


function make_suffix_array:
	new_string = query + $
	let suffix_list be a list
	let suffixes be a list
	for i in length of new_string do
		slice off first character of new_string
		append to suffixes
		append i to suffix_list
	lexicographical sort suffixes			#this creates the upper triangle of the matrix without matrix formatting; keep indices with suffixes

function BW_transform:
	let new_string be query + $
	let suffix_array be the indices sorted according to the suffixes
	let suffixes be the sorted suffixes
	let BWT be an array for characters
	for i in length of suffix_list do
		q = suffix_array[i] - 1				#pointer points us to the correct index in the original string shifted by 1 for $
		append new_string[q] to BWT			#retrieved the correct corresponding character in the original

function search_BWT:
	let left_col be the suffix_list aligned with suffixes from make_suffix_array
	let right_col be the BWT characters from BW_transform
	left suffix_array be the original indices gathered from make_suffix_array
	let query be the search query
	let reference be new_string from earlier

	create "count array"					#this is the array of first-occurrence indices in left_col
	create "occurrence array"				#this is the cumulative array of occurrence indices in right_col

	#=== search code ===
	upper = len(suffix_array) - 1			#pointer for the upper search index
	lower = 0								#pointer for the lower search index

	for character in reversed query do
		i = "count array"[character]

		if lower <= 0:						#used less than or equal to because of off-by-one issues
			lower = i
		else
			lower = i + "occurrence array"[character][lower - 1]

		upper = i + "occurrence array"[character][upper] - 1

	query_index = suffix_array[lower: upper + 1]		#just slice the original suffix array to get the indices of the original string for the query
														#this is the index in the original string of the query
	
		
'''implementation notes'''


orig = original string
query = query string

n = len(orig)
orig_indices = `[0....n]`

suffixes_unsorted = `[orig[i:] for i in orig_indices]` (if we wanted full rotations then it's `orig[i:]+orig[:i]`)

suffix_array = orig_indices sorted by suffixes_unsorted as the index (will be in lexicographic)

bwt = built letter-wise: for i in suffix_array, the next letter is `orig[i-1]`

----

make count dictionary C where `C[c]` gives the first row in S1 where c (a character) appears (all suffixes (rows) whose value in S1 < c will come before `C[c]`)

One way to build C would be:

1. create counts = {c:n} where c is the character and n is the number of times it appears in bwt (or S1 or orig, it's equivalent)

2. create C from smallest to largest lexicographic character:
    1. total = 0
    2. for c in sorted(counts): `C[c] = total, total += counts[c]`

----

define what I will call a function Occ(c,i).

- when i=0, Occ(c,i) = 0

- when i>0, Occ(c,i) = how many times c has appeared in `bwt[0...i]`, <u>i included</u>.

----

Search pseudocode:

prior operations:

- suffix_array was computed first
- the bwt was created using the suffix_array
- Occ and C were created using bwt
- bwt never used again

assuming: 
- Occ(c,i) returns how many times c has occured in bwt[0...i] for i > 0
	(it can be a dict computed once instead of a function if we explicitly assign for i =< 0, i just think of it as a function)
- C[c] returns how many times c appears in the string

steps:
1. start with every suffix in our window

lower = 0 
upper = n - 1

2. repeatedly update for longer and longer suffixes

for c in reversed(query):
	earliest_c = C[c]
	
	if lower =< 0 : 
			lower = earliest_c
		else : 
			lower = earliest_c + Occ(c,lower-1)
	
	upper = earliest_c + Occ(c,upper) - 1
	
	if lower > upper: no match found

3. return positions in orig
   
   return suffix_array[lower:upper+1]
```

# Successes
A major success was the teams ability to talk through the implementation, ensuring everyone was on the same page and able to understand what each line of code was doing. This in turn led to effective troubleshooting, being able to talk through the issues and identifying where execution was going wrong. In the end, we were able to successfully implement the algorithm, which felt like another big success to us.

# Struggles
We found most of the algorithms complexity lied within its implementation, rather than our understanding of it. The most significant hurdle was navigating the indexing challenges that naturally accompanied the algorithms implementation for string-matching. Most of the debugging and troubleshooting occured within the match finding algorithm, where we were finding that we were consistently off from identifying the position of the query by one index. It required a lot of trial and error to figure this out

# Personal Reflections
## Group Leader
Spencer Todd: I thought this weeks project walked the line between fun and challenging well. A major success for me was the ability to walk through and talk out the logic for this algorithm with Linh and Eric, as it was easy to get lost in the implementation. I struggled with the aspects of implementaion that relied heavy on indexing, initially struggling with string matching and understanding conceptually the properties of BWT that allow us to shift search windows in the BWT to find the position in the original string of our queried sub-string.

## Other member
Other members' reflections on the project

Eric Arnold: This was a fun algorithm. The concept of a transform in discrete space is difficult to wrap your head around at first. Typically transformations like this use eigenvalues and vectors (PCA, TSVD) or some kind of gradient optimization. This was the first discrete transformation I'd encountered. I think the key to understanding this one comes in with the search function. The role of upper and lower in pointing to indices in the count and occurrence arrays in reconstructing the original string reveals the fact that the indices of the string have just been remapped. The applications in genomics are clear, where you often encounter multiple repeats in the characters. It would be interesting to see how this algorithm has been applied elsewhere. I can imagine that some verisions of probabilistic models with discrete hidden states can encode large amounts of information using this techinique.

Linh: I enjoyed wrapping my head (cyclically and exposing all possible substrings) around the Burrows–Wheeler transform and the properties of lexicographic sorting / suffix arrays. It is one of those things where the more you understand it the simpler it becomes (not true for a lot of topics). Not just easier but actually simpler, as once the central logic clicks everything else follows naturally, and once every step and term's role is understood it is much easier to debug things like indexing because you know what it's "supposed to be", but before that point it was hard to identify where to start looking for that understanding. As is often the case when studying algorithms, explicit printing of intermediate values and organizing the script into discrete steps was very helpful; combined with theoretical understanding, that makes it easy to at least immediately identify what line of code an error is coming from, even before figuring out why that line of code was incorrect. I can also now much more clearly see why BWT is used for compression of bioinformatics sequence data.

# Generative AI Appendix
As per the syllabus
