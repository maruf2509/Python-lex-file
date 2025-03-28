﻿# Python-lex-file
%{
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
int yylineno = 1;  // Line number tracking


int is_similar_to_keyword(const char* str) {
    char *keywords[] = {
        "False", "None", "True", "and", "as", "assert", "async", "await", "break",
        "class", "continue", "def", "del", "elif", "else", "except", "finally",
        "for", "from", "global", "if", "import", "in", "is", "lambda", "nonlocal",
        "not", "or", "pass", "raise", "return", "try", "while", "with", "yield"
    };
    int num_keywords = 35;
    
       for (int i = 0; i < num_keywords; i++) {
        if (abs(strlen(str) - strlen(keywords[i])) <= 2) {  // Allow 2 char difference in length
            int matches = 0;
            for (int j = 0; j < strlen(str) && j < strlen(keywords[i]); j++) {
                if (str[j] == keywords[i][j]) matches++;
            }
            
            // If more than 70% of characters match, consider it similar
            if (matches >= strlen(keywords[i]) * 0.7) return 1;
        }
    }
    return 0;
}
%}
%option noyywrap
DIGIT [0-9]
LETTER [a-zA-Z]
ID [a-zA-Z_][a-zA-Z0-9_]*
INTEGER {DIGIT}+
FLOAT {DIGIT}+\.{DIGIT}+
WHITESPACE [ \t]+
INDENT ^[ \t]+
%%  
"False"|"None"|"True"|"and"|"as"|"assert"|"async"|"await"|"break"|"class"|"continue"|"def"|"del"|"elif"|"else"|"except"|"finally"|"for"|"from"|"global"|"if"|"import"|"in"|"is"|"lambda"|"nonlocal"|"not"|"or"|"pass"|"raise"|"return"|"try"|"while"|"with"|"yield" { printf("KEYWORD: %s\n", yytext); }
">="|"<="|"=="|"!="|">"|"<" { printf("RELATIONAL OPERATOR: %s\n", yytext); }
"+"|"-"|"*"|"/"|"%"|"**"|"//" { printf("ARITHMETIC OPERATOR: %s\n", yytext); }
"="|"+="|"-="|"*="|"/="|"%="|"//="|"**="|"&="|"|="|"^="|">>="|"<<=" { printf("ASSIGNMENT OPERATOR: %s\n", yytext); }
"&"|"|"|"^"|"~"|"<<"|">>" { printf("BITWISE OPERATOR: %s\n", yytext); }
"{"|"}"|"["|"]"|"("|")"|","|";"|"."|":"|"@" { printf("SYMBOL: %s\n", yytext); }
{ID}[ \t]*"(" { char func_name[100]; sscanf(yytext, "%99[^(]", func_name); printf("FUNCTION NAME: %s\n", func_name); }
{ID} {
    char *keywords[] = {
        "False", "None", "True", "and", "as", "assert", "async", "await", "break",
        "class", "continue", "def", "del", "elif", "else", "except", "finally",
        "for", "from", "global", "if", "import", "in", "is", "lambda", "nonlocal",
        "not", "or", "pass", "raise", "return", "try", "while", "with", "yield"
    };
    int is_keyword = 0;
    for (int i = 0; i < 33; i++) { 
        if (strcmp(yytext, keywords[i]) == 0) {
            is_keyword = 1;
            break;
        }
    }
    
    if (is_keyword) {
        printf("KEYWORD: %s\n", yytext);
    } else if (is_similar_to_keyword(yytext)) {
        fprintf(stderr, "ERROR: Invalid keyword at line %d: %s\n", yylineno, yytext);
    } else {
        printf("IDENTIFIER: %s\n", yytext);
    }
}
{INTEGER} { printf("INTEGER: %s\n", yytext); }
{INTEGER}[^0-9a-zA-Z_\n ] { fprintf(stderr, "ERROR: Invalid integer format at line %d: %s\n", yylineno, yytext); }
{DIGIT}+(\.{DIGIT}+){2,} { fprintf(stderr, "ERROR: Invalid floating point number at line %d: %s\n", yylineno, yytext); }
{FLOAT} { printf("FLOAT: %s\n", yytext); }
\"([^\"\n])*\"|\'([^\'\n])*\' { printf("STRING: %s\n", yytext); }
"#".* { printf("COMMENT: %s\n", yytext); }
{INDENT} { printf("INDENT: %zu spaces/tabs\n", strlen(yytext)); }
\n { yylineno++; printf("NEWLINE\n"); }  
{WHITESPACE} { /* Ignore */ }
. { fprintf(stderr, "ERROR: Unrecognized token at line %d: %s\n", yylineno, yytext); }
%%
int main() {
    printf("Python Lexical Analyzer\n----------------------\n");

    // Directly open the input.py file
    FILE *file = fopen("input.py", "r");
    
    if (!file) { 
        perror("Error opening file");
        exit(1);
    }
    
    printf("Analyzing file: input.py\n\n");
    yyin = file; // Set the input file to yyin

    yylex(); // Start lexical analysis

    printf("\nLexical analysis complete.\n");

    return 0;
}
