# Regular Expressions

This is a brief reference guide on a few simple regular expressions that find use very often. It is based on the POSIX regex syntax.

## Basics

A regular expression (RegEx) is a representation of a finite state automaton, and is used to verify the presence of a token within a string input. That is, a RegEx encodes a pattern which you want a string to "match". RegEx-based systems are typically programmed in a highly-optimized way, which is why they are superior to other, potentially simpler, ways of pattern matching.

Example usages of RegEx are trying to locate a variable sequence in a text file, checking if a string follows a certain pattern, or finding substrings to be replaced.

### Symbols

A few symbols have special meanings in a POSIX RegEx. The most commonly used ones are:

* `[ ]` : Square brackets are used to indicate a character set. When used in a RegEx, a character set indicates a position in the pattern that may be matched to any one character in the set. For example, `[abc]` will match either `a`, `b` or `c`, but not the sequence abc.
* `.` : A dot outside a character set is a wildcard for any character except NULL and Newline. Inside a character set, it is treated as a literal dot.
* `^` : Indicates that the token must be matched from the beginning of the input string. For example, `^abc` will match `abc`, but not `dabc`.
* `$` : Indicates that the token must be matched to the end of the input string. For example, `abc$` will match `dabc`, but not `abcd`.
* `( )` : Indicates a sub-expression, which may be marked for later use. They may then be referred to by back-reference, see `/i`.
* `*` : A `*` following a symbol, character or character set indicates that it may be repeated zero or more times in the token.
* `+` : A `+` following a symbol, character or character set indicates that it may be repeated one or more times in the token.
* `?` : A `?` following a symbol, character or character set indicates that it is optional, that is, it may appear zero or one times in the token.
* `{ }` : A `{ }` after a symbol, character or character set is a precise way of determining repetition.
    * `{n}` : Indicates the atom must repeat exactly `n` times.
    * `{n,}` : Indicates the atom must repeat at least `n` times.
    * `{n,m}` : Indicates the atom must repeat at least `n` and at most `m` times.
* `/i` : Indicates a back-reference to a previous sub-expression. `i` may vary between 1 and 9, and will match the accordingly indexed sub-expression based on the order they appear in the RegEx. For example, `(a*).*/1` indicates a token that begins and ends with the same number of trailing `a`'s.
* `|` : Indicates an `OR` operation over the pattern. For example, `a|b` will match any string that contains either the letter `a`, `b` or both, regardless of order. `ab(c|d)` will match any string containing a sub-string `ab` followed by either a `c` or a `d`.

### Character sets

Character sets have their own syntax, including certain pre-defined sets (not covered here).

* `^` : Indicates a complement operation. That is, the token must match any character except those represented by the trailing sub-expression. For example, `[^ab]` must match any character except `a` or `b`
* `\-` : Indicates a range. For example, `[a-c]` must match any character between `a` and `c`; `[1-5]` must match any character between `1` and `5`.

## Examples

I (Marcelo) will provide examples based off of my own necessity. These will be real examples from applications I make. Anyone may feel free to update this section with their own examples.

### Matching any smell element in the old (BitBucket) version of ARCADE's .ser

In order to search for the starting element of any smell, I used the following RegEx:

```
<edu.usc.softarch.arcade.antipattern.detection.(Bdc|Buo|Bco|Spf)Smell>
```

This matches the opening XML element of a smell by using mostly a static string, but allowing for variation in the three-letter prefix of the Smell type.

The sub-expression being marked served another purpose: I wanted to number the smell elements to facilitate identifying which specific element was being referenced by an XML reference, such as:

```
"../../../edu.usc.softarch.arcade.antipattern.detection.BdcSmell[3]/clusters/edu.usc.softarch.arcade.facts.ConcernCluster[15]"
```

A replacement RegEx was used to replace all occurrences of the matched token with a single command in TextPad, retaining the correct smell type and numbering each type separately:

```
<edu.usc.softarch.arcade.antipattern.detection./1Smell\i>
```

This replaces `/1` with the matched sub-expression (`Bdc`, `Buo`, `Bco` or `Spf`) and places a self-incrementing number at `\i`. The self-incrementing number is, as far as I'm aware, a feature of TextPad, not POSIX RegEx.

### Swapping two XML tags that were incorrectly placed

I had a situation where the closing elements of two tags were swapped, that is, the outer element was being closed before the inner element:

```
    <clusters>
        <set>
        </clusters>
    </set>
```

The indentation of the file was also haphazard, so I could not use a static string, say, `\t\t</clusters>\n\t</set>`. In order to fix this, I used the following RegEx to locate all instances of it and replace them with the correct string:

```
</clusters>[ ]*[\n]*[ ]*</set>
```

The RegEx looks for any sub-string starting with `</clusters>` and ending with `</set>`, with any number of empty spaces and a single new-line between them.
