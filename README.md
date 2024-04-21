# CD
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
