# yippee
An interpreter written in a single C header.

Yippe is currenty [**__CLOSED SOURCE__**], As I need more things to be functional before writing proper documentation and uploading the project to GitHub.

For now, Check the barebones documentation below to learn how you can contribute.

---

...

...

...

...

...

...

...

...

...

...

...

...

...

...

...

...

...

...

...

---

# Barebones documentation for yippee

This is a repo aiming to help me (the creator) write helper functions for different object types, etc.

## Highest priority helper functions:

- `add(...)` to add numbers, join strings, concatenate arrays and join tables.
- `subtract(...)` to subtract numbers, lists and tables.
- `multiply(...)` to multiply numbers.
- `print(...)` to write any sort of expression to the console. `hello world`, `43`, `func hello(string name)`, `native func echo()`, etc.
- `dump(...)` to write the SYNTAX TREE of an input expression to the console. `STRING<"hello world">`, `FUNC<HELLO, LIST<OBJ<string, NAME<name>>>, BLOCK<...>>`, `FUNC<NATIVE>`, etc.

## SPECIFICATION

```c
const var null  = ((void *) 0);

typedef char *string;
typedef unsigned long long ull;
typedef void *var; // void pointer

typedef struct array {
    ull  size;
    var *data;
} array;

typedef struct str {
    ull   size;
    char *data;
} str;

// basically `init(array)` becomes `init_array()`
#define init(type) CAT(init_, type)()

array *init_array();
str *init_str();
array *push_array(array *arr, var value);
str *push_str(str *arr, char value);
array *pop_array(array *arr);
str *pop_str(str *arr);
array *rem_array(array *arr, ull index);
str *rem_str(str *arr, ull index);

table *init_table();
var get_value(table *table, string key);
void set_value(table *table, string key, var value);

const string EXPRESSION_TYPES[]
    = { [TYPE_EXPR_NAME] = "name",     [TYPE_EXPR_NUMBER] = "number",
        [TYPE_EXPR_STRING] = "string", [TYPE_EXPR_BOOLEAN] = "boolean",
        [TYPE_EXPR_ARRAY] = "array",   [TYPE_EXPR_TABLE] = "dict",
        [TYPE_EXPR_FUNCTION] = "func", [TYPE_EXPR_INVOCATION] = "invocation" };

#define equals(x1, x2) (strcmp(x1, x2) == 0)

const string RESERVED_WORDS[]
    = { "func", "const", "number", "string", "boolean", "array", "table", "any" };
const string STRICT_RESERVES[] = { "nil", "echo", "exit", "true", "false" };
#define length(x) (sizeof(x) / sizeof(x[ 0 ]))

typedef struct linkednode {
    ull                ID;
    var                value;
    struct linkednode *prev;
    struct linkednode *next;
} linkednode;

typedef struct linkedlist {
    ull         counter;
    ull         size;
    linkednode *head;
    linkednode *end;
} linkedlist;

// a table entry.
typedef struct tablevalue {
    string key;
    var    value;
} tablevalue;

// a table.
typedef linkedlist table;

typedef struct scopetablevalue {
    string key;
    EXPR  *value;
} scopetablevalue;

typedef struct scopetablenode {
    ull                    ID;
    scopetablevalue       *value;
    struct scopetablenode *prev;
    struct scopetablenode *next;
} scopetablenode;

typedef struct scopetable {
    ull             counter;
    ull             size;
    scopetablenode *head;
    scopetablenode *end;
} scopetable;

// defines a linkedlist node.
typedef struct scopenode {
    ull               ID;
    scopetable       *value;
    struct scopenode *prev;
    struct scopenode *next;
} scopenode;

typedef struct scope {
    ull        counter;
    ull        size;
    scopenode *head;
    scopenode *end;
} scope;

typedef struct EXPR_NAME { // name, hello_world
    str *name;
} EXPR_NAME;
typedef struct EXPR_NUMBER { // 2, 7.4, 900, -303
    double value;
} EXPR_NUMBER;
typedef struct EXPR_STRING { // 'hello', "world!"
    str *value;
} EXPR_STRING;
typedef struct EXPR_BOOLEAN { // true, false
    bool value;
} EXPR_BOOLEAN;

typedef struct EXPR_ARRAY { // [2, 'hello', false]
    array *value;
} EXPR_ARRAY;

typedef struct EXPR_TABLE { // { hello: 'world!', balls: 2 }
    table *entries;
} EXPR_TABLE;

typedef struct EXPR_INVOCATION { // 4 -> echo
    enum { TYPE_EXPR_INVOCATION_NAMED, TYPE_EXPR_INVOCATION_LAMBDA } type;
    EXPR_NAME     *name;
    TREE_FUNCTION *expression;
    EXPR          *input;
} EXPR_INVOCATION;

typedef struct EXPR {
    enum {
        TYPE_EXPR_NAME,
        TYPE_EXPR_NUMBER,
        TYPE_EXPR_STRING,
        TYPE_EXPR_BOOLEAN,
        TYPE_EXPR_ARRAY,
        TYPE_EXPR_TABLE,
        TYPE_EXPR_FUNCTION,
        TYPE_EXPR_INVOCATION
    } type;
    union {
        EXPR_NAME       *name;
        EXPR_NUMBER     *number;
        EXPR_STRING     *string;
        EXPR_BOOLEAN    *boolean;
        EXPR_ARRAY      *array;
        EXPR_TABLE      *table;
        TREE_FUNCTION   *function;
        EXPR_INVOCATION *invocation;
    } value;
} EXPR;

typedef enum OBJ_TYPE { // any, string, array, ...
    TYPE_NUMBER,
    TYPE_STRING,
    TYPE_BOOLEAN,
    TYPE_ARRAY,
    TYPE_TABLE,
    TYPE_FUNCTION,
    TYPE_ANY,
    TYPE_VOID
} OBJ_TYPE;

// TREE TYPES

typedef struct TREE_BLOCK { // { -x 4 -y 56 'hello' -> echo }
    EXPR_ARRAY(TREE) * exprarray;
} TREE_BLOCK;

typedef struct TREE_VARIABLE { // -x ['hello, ' 'world!'] -> cat
    EXPR_NAME *name;
    EXPR      *expression;
} TREE_VARIABLE;

typedef struct TREE_CONST { // const x 45
    EXPR_NAME *name;

    EXPR *value;
} TREE_CONST;

typedef struct FUNC_ARG { // any x, string text
    EXPR_NAME *name;
    OBJ_TYPE   type;
} FUNC_ARG;

typedef EXPR *(*native_function)(EXPR_ARRAY *input, scope *scope);

typedef struct TREE_FUNCTION { // func giveFiveAndEight() { [5, 8]; }
    EXPR_NAME *name;
    EXPR_ARRAY(FUNC_ARG) * args;
    enum { FUNCTION_NAMED, FUNCTION_LAMBDA } naming;
    enum { BODY_EXPR, BODY_TREE, NATIVE } type;
    union {
        EXPR           *expr;
        struct TREE    *tree;
        native_function native;
    } body;
} TREE_FUNCTION;

typedef struct TREE { // const x 7; func test(any x) { x -> echo; }
    enum {
        TREE_TYPE_BLOCK,
        TREE_TYPE_VARIABLE,
        TREE_TYPE_FUNCTION,
        TREE_TYPE_CONST,
        TREE_TYPE_INVOCATION
    } type;
    union {
        TREE_BLOCK      *block;
        TREE_VARIABLE   *variable;
        TREE_FUNCTION   *function;
        TREE_CONST      *constant;
        EXPR_INVOCATION *invocation;
    } value;
} TREE;

var smalloc(ull size);
var srealloc(var ptr, ull size);
string format(string fmt, ...);
#define make(type) smalloc(sizeof(type))

typedef EXPR *(*native_function)(EXPR_ARRAY *input, scope *scope);
```

### HELPER FUNCTIONS

Helper functions should return a valid `EXPR*` and take in an `EXPR_ARRAY*` and a `scope*`. A scope is a LinkedList of tables which acts like the stack.

Example:

```c
EXPR *add_numbers(EXPR_ARRAY *arr, scope *s) {
    EXPR *e                = make(EXPR);
    e->type                = TYPE_EXPR_NUMBER;
    e->value.number        = make(EXPR_NUMBER);
    e->value.number->value = 0;

    for (ull i = 0; i < arr->value->size; i++) {
        EXPR *cur = arr->value->data[ i ];
        if (cur->type != TYPE_EXPR_NUMBER) {
            throw(format(
                "native function [add_numbers] expected an array of numbers, but got type [%s] as "
                "the [%llu]th element.",
                EXPRESSION_TYPES[ cur->type ],
                i + 1));
        }

        e->value.number->value += cur->value.number->value;
    }

    return e;
}
```

Please note that this function is not generic enough and it is on my priority list to make it also apply to strings, lists, etc.
