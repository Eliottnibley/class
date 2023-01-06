Assignment 1: Writing some JPL
==============================

Your first assignment is to build a lexical analyzer (a "lexer" or a
"tokenizer") for JPL. Its job is to turn an arbitrary file into one
of:

- A list of tokens
- A lexical error

You will be tested on a collection of lexer test cases created by the
instructors.

# Your Assignment

The criteria for tokenization are found in the [lexical
syntax][lex-spec] part of the JPL specification. The full list of
tokens you should support is:

[lex-spec]: https://github.com/utah-cs4470-sp21/jpl/blob/main/spec.md#lexical-syntax

    ARRAY ASSERT BOOL ELSE FALSE FLOAT FN IF IMAGE INT LET PRINT
    READ RETURN SHOW SUM THEN TIME TO TRUE TYPE WRITE
    COLON LCURLY RCURLY LPAREN RPAREN COMMA LSQUARE RSQUARE EQUALS
    NEWLINE END_OF_FILE INTVAL FLOATVAL VARIABLE OP

Note that some tokens, like `ARRAY` or `LCURLY` correspond to only one
possible string, while other tokens are non-trivial and correspond to
multiple possible strings (`INTVAL`, `OP`).

Your lexer must read in a JPL file and output the sequence of tokens
that it contains. When lexing is successful, a special `END_OF_FILE`
token should be the last element of the list. It does not correspond
to any actual text in the input file, but rather serves as a sentinel
so that your parser will not have to keep checking for walking off the
end of the list of lexemes.

Specifically, you must start to implement the [JPL command line
interface][iface]. Your compiler must print `Compilation successful`
to the standard output when lexing is successful and `Compilation
failed` when a lexer error is encountered. We will use these features
when testing your code. Of course you do not yet need to support the
other command line flags.
  
[iface]: https://github.com/utah-cs4470-sp23/jpl/blob/main/spec.md#jpl-compiler-command-line-interface

Additionally, your compiler must support the "lex-only" mode triggered
by the `-l` command line flag. With this flag, your compiler should
print the lexed tokens, one per line, in a format like this:

    FN 'fn'
    VARIABLE 'inc'
    LPAREN '('
    VARIABLE 'n'
    COLON ':'
    VARIABLE 'int'
    RPAREN ')'
    COLON ':'
    VARIABLE 'int'
    LCURLY '{'
    NEWLINE
    RETURN 'return'
    INTVAL '1'
    OP '+'
    VARIABLE 'n'
    NEWLINE
    RCURLY '}'
    NEWLINE
    PRINT 'print'
    STRING '"hello!"'
    NEWLINE
    SHOW 'show'
    VARIABLE 'inc'
    LPAREN '('
    INTVAL '33'
    RPAREN ')'
    NEWLINE
    END_OF_FILE

Each token prints its name and its contents, except for `NEWLINE` and
`END_OF_FILE` tokens. Since your code will be auto-graded, you must
match this output format exactly.

**Important note:** You should not try to enforce tricky properties,
such as integers or float values being out of range, in your lexer.
These properties are easy to enforce during parsing.

# Handling Whitespace

Make sure to handle white space correctly:

 - Space characters can separate tokens, but don't generate tokens of
   their own.
 
 - Comments also do not turn into tokens, they are simply eaten by the
   lexer. Be particularly careful to avoid emitting multiple
   consecutive `NEWLINE` tokens in the presence of comments in the
   input.

 - Newlines do turn into tokens, and consecutive newlines in the input
   should be collapsed into a single `NEWLINE` token, even if
   separated by comments or whitespace. Your lexer should never
   produce multiple consecutive `NEWLINE` tokens.
  
 - Line continuations (backslash at the end of a line) likewise does
   not produce a token. The newline that follows it also must not
   produce a token.
   
 - All other whitespace characters are illegal, and you must produce a
   lexer error upon encountering one in a file. As you do your own
   tests, be extra careful that you are saving files with Unix-style
   line endings (LF). Windows-style line endings (CR LF) are invalid
   in JPL.
   
# Implementation Suggestions

The list of tokens should use a suitable list or array data structure
in your compiler implementation language. For example, in C++ it might be:

```
std::vector<lexeme> tokens;
```

where a lexeme is:

```
struct lexeme {
  tok t;
  int start;
  std::string text;
};
```

Where `tok` is an enum type, `start` is the byte position in the input
file where the lexeme was found, and `text` is the lexeme itself.

Note that the `start` field is not required in this assignment, but is
essential to producing good error messages. Since the only user of
your compiler is you, it's an investment worth making. We recommend
saving the name and contents of input file in memory, from which you
can reconstruct the line and column number from any byte position.

We recommend using regular expressions to define the complex tokens
`INTVAL`, `FLOATVAL`, `VARIABLE`, and `STRING`, as well as whitespace.
C++, Python, and Java all have decent regular expression engines that
support this syntax:

- [C++](http://www.cplusplus.com/reference/regex/ECMAScript/)
- [Java](https://docs.oracle.com/javase/tutorial/essential/regex/)
- [Python](https://docs.python.org/3/library/re.html)

The other token types are, however, probably easier to support using
direct string matches. That makes sure you don't run into tricky bugs
related to escaping special characters in regular expressions.

Rigorously think through both what strings a token should match as
well as which tokens it should *not* match. For example, make sure
`1.0.0` and `.` are not considered valid `FLOATVAL`s! In some cases,
you can use order to help; for example, instead of writing a
`VARIABLE` regular expression that excludes all keywords, it may be
easier to match keywords first, and only match variables if that
fails.

# Testing your code

The JPL interpreter in the "[Releases][releases]" supports the `-l`
operation you are being asked to implement. You can run it on any test
file to see the correct lexing of that file:

    ~/Downloads $ ./jplc-macos -l ~/jpl/examples/gradient.jpl
    NEWLINE
    FN 'fn'
    VARIABLE 'gradient'
    LPAREN '('
    VARIABLE 'i'
    COLON ':'
    INT 'int'
    COMMA ','
    VARIABLE 'j'
    COLON ':'
    ...

Naturally, this JPL interpreter is a program and can have bugs. If you
think you've found one, contact the instructors on Discord.

[releases]: https://github.com/utah-cs4470-sp23/class/releases

For larger programs, it can be tedious to compare the list of tokens
by hand. Instead, save each output to a file and use `diff` to
compare:

    diff wrong-output.txt right-output.txt
    
This will print all the lines that differ; you can use various `diff`
flags to get more context for even larger files.

Once things are working, push everything to your repository. Make sure
you can run your compiler like so:

    make run TEST=input.jpl

Here `input.jpl` is a file that we will supply. Your makefile is
responsible for passing the `-l` flag to your compiler. Additionally,
the `make compile` command must complete successfully.

You can find the tests and expect outputs [in the auto-grader
repository](https://github.com/utah-cs4470-sp23/grader/tree/main/hw2).
We are providing two scripts that you can use to test your lexer is
working properly. These are the same scripts that that the auto-grader
uses to grade your lexer.

The first script, `test-lexer1` compares the output of your compiler,
in lexer mode, against a reference lexer. It complains if your output
does not exactly match the reference output. It uses the `-l` flag
mentioned in the JPL specification. Run it like this, but instead of
`../jdr/Makefile` you should specify the location of your Makefile:

```
Johns-MacBook-Pro:tests johnregehr$ ./test-lexer1 ../jdr/Makefile
/Users/johnregehr/compiler-class/tests/lexer-tests1
/Users/johnregehr/compiler-class/tests/lexer-tests1/000.jpl : pass
/Users/johnregehr/compiler-class/tests/lexer-tests1/001.jpl : pass
/Users/johnregehr/compiler-class/tests/lexer-tests1/002.jpl : pass
/Users/johnregehr/compiler-class/tests/lexer-tests1/003.jpl : pass
/Users/johnregehr/compiler-class/tests/lexer-tests1/004.jpl : *** /Users/johnregehr/compiler-class/tests/lexer-tests1/004.jpl.my-output	2021-01-22 11:50:39.000000000 -0700
--- /Users/johnregehr/compiler-class/tests/lexer-tests1/004.jpl.output	2021-01-22 11:43:08.000000000 -0700
***************
*** 2,8 ****
  FN 'fn'
  VARIABLE 'a'
  LPAREN '('
! VARIABLE 'b '
  LSQUARE '['
  VARIABLE 'c'
  RSQUARE ']'
--- 2,8 ----
  FN 'fn'
  VARIABLE 'a'
  LPAREN '('
! VARIABLE 'b'
  LSQUARE '['
  VARIABLE 'c'
  RSQUARE ']'
Johns-MacBook-Pro:tests johnregehr$ 
```

In this example run, the output of our lexer matched the reference
output for `000.jpl` through `003.jpl` but then did not match for
`004.jpl`. The difference is shown with the correct output first and
your output second. In this case the bug is that my lexer accidentally
captured an extra space character after the text for the variable `b`.

Run the second test script, `test-lexer2`, the same way. This one does
not look at the actual lexer output, but rather simply checks if
lexing succeeds (because the input can be lexed) or fails (because the
input does not meet the requirements for lexing that are described in
the JPL specification).

# Submission and grading

This assignment is due Friday Jan 20.

We are happy to discuss problems and solutions with you on Discord, in
office hours, or by appointment.

Your compiler must be runnable as described in the [Testing your code]
section. If the auto-grader cannot run your code, you will not receive
credit. The auto-grader output is available to you at any time, as
many times as you want. Make use of it.

The rubrik is:

| Weight | Function              |
|--------|-----------------------|
| 10%    | Keywords & operators  |
| 15%    | Variables             |
| 15%    | Integers              |
| 15%    | Strings               |
| 20%    | Floats                |
| 25%    | Whitespace & comments |

Your solutions will be auto-graded. The auto-grader will use Github
Actions and run on Ubuntu using the most recent release of the
interpreter described above.
