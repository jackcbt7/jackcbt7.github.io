---
layout: post
title: Lex and Yacc, chapter 1
---

The first section with
```
%{


%}
```

is called definition section, it introduces any initial C program code we want copied into the final program


The next section is the *rules* section. Each rule is made up of two parts: a *pattern* and an *action*, separated by whitespace. The lexer that lex generates will execute the action when it recognizes the pattern. These patterns are UNIX-style regular expresions, a slightly extended version of the same expressions used by tools such as *grep*, *sed* and *ed*.

The first rule in our example (ch1-02.l) is the following:
```
[\t ]+  /* ignore whitespace */;
```
The `[]` indicate that any one of the characters within the brackets matches the pattern. The `+` means that the pattern matches one or more consecutive copies of the subpattern that precedes the plus.
The second part of the rule, the *action*, is simply a semicolon, a do-nothing C statement. Its effect is to ignore the input.

The next set of rules use `|` (vertical bar) action. This is a special action that means to use the same action as the next pattern, so all of the verbs use the action specified for the last one.

The array `yytext` contains the text that matched the pattern. This action will print the recognized verb followed by the string `": is a verb\n"`.

The last two rules are:
The pattern `[a-zA-Z]+` is a common one: it indicates any alphanumeric string with at least one character. The `-` character has special meaning when used inside square brackets: it denotes a range of characters beginning with the character to the left of the `-` and ending with the character to its right.

The two rules that make our lexer work are:

1. Lex patterns only match a given input character or string once.
2. Lex executes the action for the longest possible match for the current input. Thus, lex would see `island` as matching our all-inclusive rule because that was a longer match than `is`.

The last line is the default case. The special character `.` (perioed) matches any single character other than a newline, and `\n` matches a newline character. The special action `ECHO` prints the matched pattern on the output, copying any punctuation or other characters.

The final section is the *user subroutines section*, which can consist of any legal C code. Lex copies it to the C file after the end of the lex generated code.

### Grammars

To recognize specific sequences of tokens and perform appropriate actions, traditionally a description of such a set of actions is known as a *grammar*.


```
\.\n    { state = LOOKUP;
            return 0;
        }
```

The backslash in front of the period quotes the period, so this rule matches a period followed by a newline. The other change we made to our lexical analyzer was to omit the `main()` routine as it will now be provided within the parser.

## A Yacc Parser
### The Rules Section

The rules section describes the actual grammar as a set of *production rules* or simply *rules*. Each rule consists of a single name on the left-hand side of the `:` operator, a list of symbols and action code on the right-hand side, and a semicolon indicating the end of the rule. By default, **the first rule is the highest-level rule**. That is, the parser attempts to find a list of tokens which match this initial rule, or more commonly, rules found from the initial rule. The expression on the right-hand side of the rule is a list of zero or more names. A typical simple rule has a single symbol on the right-hand side as in the object rule which is defined to be a NOUN. The symbol on the left-hand side of the rule can then be used like a token in other rules. From this, we build complex grammars.

```
sentence: subject VERB object { printf("Sentence is valid\n"); }
```
The action part of the block consists of a C block, beginning with `{` and ending with `}`. The parser executes an action at the end of a rule as soon as the rule matches. In our **sentence** rule, the action reports that we've successfully parsed a sentence. Since **sentence** is the top level symbol, the entire input must match a **sentence**. The parser returns to its caller, in this case the main program, when the lexer reports the end of the input. Subsequent calls to `yyparse()` reset the state and begin processing again.

When the parser sees invalid tokens, it calls `yyerror()`, which we provide in the user subroutines section, and then recognizes the special rule error. You can provide error recovery code that tries to get the parser back into a state where it can continue parsing. If error recovery fails or, as is the case here, there is no error recovery mode, `yyparse()` returns to the caller after it finds an error.


We introduced *recursion* into this grammar. Recursion, in which a rule refers directly or indirectly to itself, is a powerful tool for describing grammars, and we use the technique in nearly every yacc grammar we write.

### Running Lex and Yacc

```bash
$ lex ch1-n.l
$ yacc -d ch1-m.y
$ cc -c lex.yy.c y.tab.c
$ cc -o example-m.n lex.yy.o y.tab.o -ll
```

The first line runs lex over the lex specification and generates a file, *lex.yy.c*, which contains C code for the lexer. In the second line, we use yacc to generate both `y.tab.c` and `y.tab.h` (the latter is the file of token definitions created by the `-d` switch).
