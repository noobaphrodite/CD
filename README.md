# CD
**FLEX Program**
%{
#include <stdio.h>
int word_count = 0;
int vowel_count = 0;
%}

DIGIT   [0-9]
LETTER  [a-zA-Z]

%%
{DIGIT}+    { printf("Found a number: %s\n", yytext); }
{LETTER}+   { 
                printf("Found a word: %s\n", yytext);
                word_count++; 
                int i;
                for (i = 0; yytext[i] != '\0'; i++) {
                    char c = yytext[i];
                    if (c == 'a' || c == 'e' || c == 'i' || c == 'o' || c == 'u' ||
                        c == 'A' || c == 'E' || c == 'I' || c == 'O' || c == 'U') {
                            vowel_count++;
                        }
                }
            }
[ \t\n]+    ; // Ignore whitespace
.           { printf("Found a special character: %c\n", *yytext); }
%%

int main() {
    yylex();
    printf("Total words: %d\n", word_count);
    printf("Total vowels: %d\n", vowel_count);
    return 0;
}

**Implementation of Scanner**
%{
#include <stdio.h>
%}

DIGIT       [0-9]
LETTER      [a-zA-Z]
IDENTIFIER  ({LETTER}({LETTER}|{DIGIT})*) 

%%
"if"        { printf("Keyword: if\n"); }
"else"      { printf("Keyword: else\n"); }
"while"     { printf("Keyword: while\n"); }
"for"       { printf("Keyword: for\n"); }
"int"       { printf("Keyword: int\n"); }
"float"     { printf("Keyword: float\n"); }
"char"      { printf("Keyword: char\n"); }

{IDENTIFIER}    { printf("Identifier: %s\n", yytext); }

{DIGIT}+        { printf("Number: %s\n", yytext); }

"+" | "-" | "*" | "/" | "=" | "==" | "!=" | ">" | "<" | ">=" | "<=" | "&&" | "||" | "!" | ";" | "," | "(" | ")" | "{" | "}" | "[" | "]" | "." | "->"
                { printf("Symbol: %s\n", yytext); }

[ \t\n]         ; // Ignore whitespace
.               { printf("Unknown token: %s\n", yytext); }

%%

int main() {
    yylex();
    return 0;
}
**PREDICTIVE PARSING**
%{
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Function to perform predictive parsing
void parse(const char *input_string);

// Function to print parsing error
void parsing_error();

// Function to print parsing success
void parsing_success();

// Parsing table
char *parsing_table[4][4] = {
    {"A", "A", "A", ""},
    {"", "", "", ""},
    {"", "", "", ""},
    {"B", "", "", ""}
};

// Token types
enum TokenType { IDENTIFIER, NUMBER, KEYWORD, OPERATOR, SEMICOLON, UNKNOWN };

// Function to get token type
enum TokenType get_token_type(const char *token);

// Function to get column index in parsing table
int get_column_index(enum TokenType token_type);

%}

%option noyywrap

DIGIT       [0-9]
LETTER      [a-zA-Z]

%%

[a-zA-Z]+   { printf("IDENTIFIER: %s\n", yytext); parse(yytext); }
[0-9]+      { printf("NUMBER: %s\n", yytext); parse(yytext); }
"if"        { printf("KEYWORD: if\n"); parse("if"); }
"else"      { printf("KEYWORD: else\n"); parse("else"); }
"while"     { printf("KEYWORD: while\n"); parse("while"); }
"for"       { printf("KEYWORD: for\n"); parse("for"); }
"+"         { printf("OPERATOR: +\n"); parse("+"); }
"-"         { printf("OPERATOR: -\n"); parse("-"); }
"*"         { printf("OPERATOR: *\n"); parse("*"); }
"/"         { printf("OPERATOR: /\n"); parse("/"); }
"="         { printf("OPERATOR: =\n"); parse("="); }
";"         { printf("SEMICOLON\n"); parse(";"); }
[ \t\n]     ; // Ignore whitespace
.           { printf("UNKNOWN: %s\n", yytext); parsing_error(); }

%%

int main() {
    const char *input_string = "if (x < 10) { x = x + 1; }";
    parse(input_string);
    return 0;
}

// Function to perform predictive parsing
void parse(const char *input_string) {
    // Stack to simulate the parsing process
    char stack[100];
    int top = -1;

    // Push the start symbol onto the stack
    stack[++top] = 'S';

    // Initialize input index
    int input_index = 0;

    // Loop until the stack is empty
    while (top >= 0) {
        // Get the top of the stack
        char current_symbol = stack[top];
        enum TokenType current_token = get_token_type(input_string[input_index]);

        // If current symbol is a terminal
        if (current_symbol == current_token) {
            // Pop the stack
            top--;
            // Move to the next token in the input
            input_index++;
        }
        // If current symbol is non-terminal
        else {
            // Get the column index in the parsing table
            int col_index = get_column_index(current_token);
            if (col_index == -1) {
                parsing_error();
                return;
            }

            // Get the production rule from the parsing table
            char *production = parsing_table[current_symbol - 'S'][col_index];
            if (production == NULL || strcmp(production, "") == 0) {
                parsing_error();
                return;
            }

            // Pop the stack
            top--;

            // Push the production rule onto the stack in reverse order
            int len = strlen(production);
            for (int i = len - 1; i >= 0; i--) {
                stack[++top] = production[i];
            }
        }
    }

    parsing_success();
}

// Function to print parsing error
void parsing_error() {
    printf("Parsing Error!\n");
}

// Function to print parsing success
void parsing_success() {
    printf("Parsing Successful!\n");
}

// Function to get token type
enum TokenType get_token_type(const char *token) {
    if (strcmp(token, "if") == 0 || strcmp(token, "else") == 0 || 
        strcmp(token, "while") == 0 || strcmp(token, "for") == 0) {
        return KEYWORD;
    } else if (strcmp(token, "+") == 0 || strcmp(token, "-") == 0 ||
               strcmp(token, "*") == 0 || strcmp(token, "/") == 0 ||
               strcmp(token, "=") == 0) {
        return OPERATOR;
    } else if (strcmp(token, ";") == 0) {
        return SEMICOLON;
    } else if (isdigit(token[0])) {
        return NUMBER;
    } else {
        return IDENTIFIER;
    }
}

// Function to get column index in parsing table
int get_column_index(enum TokenType token_type) {
    switch (token_type) {
        case IDENTIFIER:
            return 0;
        case NUMBER:
            return 1;
        case KEYWORD:
            return 2;
        case OPERATOR:
            return 3;
        case SEMICOLON:
            return 4;
        default:
            return -1;
    }
}


**<IF-ELSE>**
%{
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static int label_counter = 0;

char* generate_label() {
    char* label = (char*)malloc(sizeof(char) * 10);
    sprintf(label, "L%d", label_counter++);
    return label;
}

extern int yylex();
extern int yyparse();
extern int yyerror(const char*);
extern FILE* yyin;
%}

%token NUMBER
%token PLUS MINUS TIMES DIVIDE LPAREN RPAREN
%token IF ELSE WHILE

%%

program: statement_list
       ;

statement_list: statement
              | statement_list statement
              ;

statement: expression { printf("Intermediate code: t := %d\n", $1); }
         | if_statement
         | while_statement
         ;

if_statement: IF LPAREN condition RPAREN statement { 
                 char *label = generate_label();
                 printf("Intermediate code: if %s goto %s\n", $3, label);
                 printf("Intermediate code: t := 1\n");
                 printf("Intermediate code: %s:\n", label);
                 free(label);
              }
            | IF LPAREN condition RPAREN statement ELSE statement { 
                 char *label1 = generate_label();
                 char *label2 = generate_label();
                 printf("Intermediate code: if %s goto %s\n", $3, label1);
                 printf("Intermediate code: goto %s\n", label2);
                 printf("Intermediate code: %s:\n", label1);
                 printf("Intermediate code: t := 1\n");
                 printf("Intermediate code: %s:\n", label2);
                 free(label1);
                 free(label2);
              }
            ;

while_statement: WHILE LPAREN condition RPAREN statement { 
                    char *label1 = generate_label();
                    char *label2 = generate_label();
                    printf("Intermediate code: %s:\n", label1);
                    printf("Intermediate code: if %s goto %s\n", $3, label2);
                    printf("Intermediate code: goto %s\n", label1);
                    printf("Intermediate code: %s:\n", label2);
                    free(label1);
                    free(label2);
                }
               ;

expression: term
          | expression PLUS term { $$ = $1 + $3; }
          | expression MINUS term { $$ = $1 - $3; }
          ;

term: factor
    | term TIMES factor { $$ = $1 * $3; }
    | term DIVIDE factor { $$ = $1 / $3; }
    ;

factor: NUMBER { $$ = $1; }
       | LPAREN expression RPAREN { $$ = $2; }
       ;

condition: expression
         | expression ">" expression { $$ = ($1 > $3) ? 1 : 0; }
         | expression "<" expression { $$ = ($1 < $3) ? 1 : 0; }
         | expression ">=" expression { $$ = ($1 >= $3) ? 1 : 0; }
         | expression "<=" expression { $$ = ($1 <= $3) ? 1 : 0; }
         | expression "==" expression { $$ = ($1 == $3) ? 1 : 0; }
         | expression "!=" expression { $$ = ($1 != $3) ? 1 : 0; }
         ;

%%

int main(int argc, char* argv[]) {
    if (argc < 2) {
        printf("Usage: %s <input_file>\n", argv[0]);
        return 1;
    }
    
    FILE* input_file = fopen(argv[1], "r");
    if (!input_file) {
        perror("Error opening file");
        return 1;
    }

    yyin = input_file;
    yyparse();
    fclose(input_file);

    return 0;
}


**<TOP-DOWN>**
**<lex>**
%{
#include "parser.tab.h"
%}

%%

[0-9]+             { yylval.num = atoi(yytext); return NUMBER; }
"+"                { return PLUS; }
"-"                { return MINUS; }
"*"                { return TIMES; }
"/"                { return DIVIDE; }
"("                { return LPAREN; }
")"                { return RPAREN; }
\n                 ; // Ignore newline
[ \t]              ; // Ignore whitespace
.                  ; // Ignore other characters

%%

int yywrap() {
    return 1;
}

**<bison>**
%{
#include <stdio.h>
#include <stdlib.h>

extern int yylex();
extern int yyparse();
extern int yylval;
extern FILE* yyin;

int yyerror(const char* msg) {
    fprintf(stderr, "Error: %s\n", msg);
    return 1;
}

%}

%token NUMBER
%token PLUS MINUS TIMES DIVIDE LPAREN RPAREN

%%

program: expression { printf("Result: %d\n", $1); }
       ;

expression: term
          | expression PLUS term { $$ = $1 + $3; }
          | expression MINUS term { $$ = $1 - $3; }
          ;

term: factor
    | term TIMES factor { $$ = $1 * $3; }
    | term DIVIDE factor { $$ = $1 / $3; }
    ;

factor: NUMBER { $$ = $1; }
       | LPAREN expression RPAREN { $$ = $2; }
       ;

%%

int main(int argc, char* argv[]) {
    if (argc < 2) {
        printf("Usage: %s <input_file>\n", argv[0]);
        return 1;
    }
    
    FILE* input_file = fopen(argv[1], "r");
    if (!input_file) {
        perror("Error opening file");
        return 1;
    }

    yyin = input_file;
    yyparse();
    fclose(input_file);

    return 0;
}

**<bottom-up>**
**<bison>**
%{
#include <stdio.h>
#include <stdlib.h>

int yylex();
void yyerror(const char*);

%}

%token NUMBER
%token PLUS MINUS TIMES DIVIDE LPAREN RPAREN

%left PLUS MINUS
%left TIMES DIVIDE

%%

program: expression { printf("Result: %d\n", $1); }
       ;

expression: expression PLUS expression { $$ = $1 + $3; }
          | expression MINUS expression { $$ = $1 - $3; }
          | expression TIMES expression { $$ = $1 * $3; }
          | expression DIVIDE expression { $$ = $1 / $3; }
          | LPAREN expression RPAREN { $$ = $2; }
          | NUMBER { $$ = $1; }
          ;

%%

int main() {
    yyparse();
    return 0;
}

void yyerror(const char* msg) {
    fprintf(stderr, "Error: %s\n", msg);
}

**<scanner>**
%{
#include <stdio.h>
%}

%%
[0-9]+             { printf("Integer: %s\n", yytext); }
[a-zA-Z][a-zA-Z0-9]* { printf("Identifier: %s\n", yytext); }
"+"                { printf("Operator: %s\n", yytext); }
"-"                { printf("Operator: %s\n", yytext); }
"*"                { printf("Operator: %s\n", yytext); }
"/"                { printf("Operator: %s\n", yytext); }
.                  ; // Ignore other characters
%%

int main() {
    yylex();
    return 0;
}

**SHIFT REDUCE PARSING**
FLEX
%{
#include "parser.tab.h" // Include the Bison-generated header file
%}

%option noyywrap

%%
[0-9]+          { yylval.num = atoi(yytext); return NUMBER; }
[+-*/()]        { return yytext[0]; }
[ \t\n]         ; // Ignore whitespace
.               { fprintf(stderr, "Invalid character: %s\n", yytext); }
%%

BISON
%{
#include <stdio.h>
extern int yylex();
extern void yyerror(const char *);
%}

%token NUMBER
%left '+' '-'
%left '*' '/'

%%
input: /* empty */ | input line;

line: expr '\n' { printf("Result: %d\n", $1); };

expr: expr '+' expr { $$ = $1 + $3; }
    | expr '-' expr { $$ = $1 - $3; }
    | expr '*' expr { $$ = $1 * $3; }
    | expr '/' expr { $$ = $1 / $3; }
    | '(' expr ')'  { $$ = $2; }
    | NUMBER        { $$ = $1; }
    ;

%%

void yyerror(const char *msg) {
    fprintf(stderr, "Error: %s\n", msg);
}

int main() {
    yyparse(); // Start parsing
    return 0;
}


**LEADING SETS OF NON TERMINALS**
%{
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
%}

%token NUMBER
%left '+' '-'
%left '*' '/'

%%

input: /* empty */ | input line;

line: expr '\n' { printf("Result: %d\n", $1); };

expr: expr '+' expr { $$ = $1 + $3; }
    | expr '-' expr { $$ = $1 - $3; }
    | expr '*' expr { $$ = $1 * $3; }
    | expr '/' expr { $$ = $1 / $3; }
    | '(' expr ')'  { $$ = $2; }
    | NUMBER        { $$ = $1; }
    ;

leading_set: /* empty */
            | leading_set non_terminal '\n'
            ;

non_terminal: 'A' { printf("Leading set of A: { +, -, *, /, (, NUMBER }\n"); }
            | 'B' { printf("Leading set of B: { +, -, *, /, (, NUMBER }\n"); }
            ;

%%

void yyerror(const char *msg) {
    fprintf(stderr, "Error: %s\n", msg);
}

int main() {
    yyparse(); // Start parsing
    return 0;
}


**Find Trailing Sets of Non-Terminals**
%{
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
%}

%token NUMBER
%left '+' '-'
%left '*' '/'

%%

input: /* empty */ | input line;

line: expr '\n' { printf("Result: %d\n", $1); };

expr: expr '+' expr { $$ = $1 + $3; }
    | expr '-' expr { $$ = $1 - $3; }
    | expr '*' expr { $$ = $1 * $3; }
    | expr '/' expr { $$ = $1 / $3; }
    | '(' expr ')'  { $$ = $2; }
    | NUMBER        { $$ = $1; }
    ;

trailing_set: /* empty */
            | trailing_set non_terminal '\n'
            ;

non_terminal: 'A' { printf("Trailing set of A: { +, -, *, /, ), NUMBER }\n"); }
            | 'B' { printf("Trailing

