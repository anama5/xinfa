# 一 Makefile

        You need a file called a *makefile* to tell `make` what to do. Most often, the makefile tells `make` how to compile and link a program.

## 1.1 Makefile 的内容

1. Makefiles contain five kinds of things:
   
   * explicit rules
     
     * An explicit rule says when and how to remake one or more files, called the rule’s targets. It lists the other files that the targets depend on, called the prerequisites of the target, and may also give a recipe to use to create or update the targets.
   
   * implicit rules
     
     * An implicit rule says when and how to remake a class of files based on their names. It describes how a target may depend on a file with a name similar to the target and gives a recipe to create or update such a target.
   
   * variable definitions
     
     * A variable definition is a line that specifies a text string value for a variable that can be substituted into the text later.
   
   * directives
     
     * A directive is an instruction for make to do something special while reading the makefile.
       
       * Reading another makefile
       
       * Deciding (based on the values of variables) whether to use or ignore a part of the makefile
       
       * Defining a variable from a verbatim string containing multiple lines
   
   * comments
     
     * ‘#’ in a line of a makefile starts a comment.
     
     * If you want a literal #, escape it with a backslash (e.g., \#).
     
     * Comments may appear on any line in the makefile, although they are treated specially in certain situations.
       
       * You cannot use comments within variable references or function calls: any instance of # will be treated literally (rather than as the start of a comment) inside a variable reference or function call.
       
       * Comments within a recipe are passed to the shell, just as with any other recipe text. The shell decides how to interpret it: whether or not this is a comment is up to the shell.
       
       * Within a define directive, comments are not ignored during the definition of the variable, but rather kept intact in the value of the variable. When the variable is expanded they will either be treated as make comments or as recipe text, depending on the context in which the variable is evaluated.

2. Splitting Long Lines
   
   * Makefiles use a “line-based” syntax in which the newline character is special and marks the end of a statement.
   
   * GNU make has no limit on the length of a statement line. However, it is difficult to read lines which are too long to display without wrapping or scrolling. So, you can format your makefiles for readability by adding newlines into the middle of a statement: you do this by escaping the internal newlines with a backslash (`\`) character.
   
   * The way in which backslash/newline combinations are handled depends on whether the statement is a recipe line or a non-recipe line.
   
   * Outside of recipe lines
     
     * backslash/newlines are converted into a single space character. all whitespace around the backslash/newline is condensed into a single space:
       
       * all whitespace preceding the backslash
       
       * all whitespace at the beginning of the line after the backslash/newline
       
       * any consecutive backslash/newline combinations
     
     * If the .POSIX special target is defined then backslash/newline handling is modified slightly to conform to POSIX.2:
       
       * whitespace preceding a backslash is not removed
       
       * consecutive backslash/newlines are not condensed
   
   * If you need to split a line but do not want any whitespace added, you can utilize a subtle trick: replace your backslash/newline pairs with the three characters dollar sign, backslash, and newline. `\newline -> $\newline`
     
     * first, make removes the backslash/newline and condenses the following line into a single space. `$\newline` -> ‘$ ’
     
     * then, make will perform variable expansion. The variable reference ‘$ ’ refers to a variable with the one-character name “ ” (space) which does not exist, and so expands to the empty string.
       
           # 1 使用 $\newline 切割成多行
           var := one$\
                   word
           
           # 2 Make 去掉 \newline
           var := one$ word
           
           # 3 Make 展开变量
            var := oneword

## 1.2 Makefile 的名称

* By default, when make looks for the makefile, it tries the following names, in order:
  
  * GNUmakefile
  
  * makefile
  
  * Makefile (Recommend)

* If you want to use a nonstandard name for your makefile, you can specify the makefile name with the ‘-f name’ or ‘--file=name’ option.
  
  * The arguments tell make to read the file `name` as the makefile.
  
  * If you use more than one ‘-f’ or ‘--file’ option, you can specify several makefiles. All the makefiles are effectively concatenated in the order specified.
  
  * The default makefile names GNUmakefile, makefile and Makefile are not checked automatically if you specify ‘-f’ or ‘--file’.

## 1.3 Including Other Makefiles

* `include filenames`
  
  1. filenames can contain shell file name patterns.
  
  2. If filenames is empty, nothing is included and no error is printed.
  
  3. Whitespace is required between include and the file names, and between file names
  
  4. If the file names contain any variable or function references, they are expanded.

. The include directive tells make to suspend reading the current makefile and read one or more other makefiles before continuing.

* If the specified name does not start with a slash (or a drive letter and colon when GNU Make is compiled with MS-DOS / MS-Windows path support), and the file is not found in the current directory, several other directories are searched. (The `.INCLUDE_DIRS` variable will contain the current list of directories that make will search for included files.)
  
  1. First, any directories you have specified with the `-I` or `--include-dir` options are searched
  
  2. Then, the following directories (if they exist) are searched, in this order
     
     * prefix/include (normally /usr/local/include)
     
     * /usr/gnu/include
     
     * /usr/local/include
     
     * /usr/include

* You can avoid searching in these default directories by adding the command line option
  -I with the special value - (e.g., `-I-`) to the command line.

* If an included makefile cannot be found in any of these directories it is not an immediately fatal error; processing of the makefile containing the include continues. Once it has finished reading makefiles, make will try to remake any that are out of date or don’t exist. Only after it has failed to find a rule to remake the makefile, or it found a rule but the recipe failed, will make diagnose the missing makefile as a fatal error.

* If you want make to simply ignore a makefile which does not exist or cannot be remade, with no error message, use the `-include` directive instead of `include`
  
  * `-include` acts like `include` in every way except that there is no error (not even a warning) if any of the filenames (or any prerequisites of any of the filenames) do not exist or cannot be remade.
  
  * For compatibility with some other make implementations, `sinclude` is another name for `-include`.

## 1.4 How make Reads a Makefile

* GNU make does its work in two distinct phases. 
  
  1. During the first phase it reads all the makefiles, included makefiles, etc. and internalizes all the variables and their values and implicit and explicit rules, and builds a dependency graph of all the targets and their prerequisites. 
  
  2. During the second phase, make uses this internalized data to determine which targets need to be updated and run the recipes necessary to update them.

. Expansion
  
  . We say that expansion is immediate if it happens during the first phase: make will expand that part of the construct as the makefile is parsed.
  
  . We say that expansion is deferred if it is not immediate. Expansion of a deferred construct part is delayed until the expansion is used: either when it is referenced in an immediate context, or when it is needed during the second phase.

### 1 Variable Assignment

## 1.5 How Makefiles Are Parsed

1. Read in a full logical line, including backslash-escaped lines 

2. Remove comments

3. If the line begins with the recipe prefix character and we are in a rule context, add the line to the current recipe and read the next line

4. Expand elements of the line which appear in an immediate expansion context

5. Scan the line for a separator character, such as ‘:’ or ‘=’, to determine whether the line is a macro assignment or a rule

6. Internalize the resulting operation and read the next line.


