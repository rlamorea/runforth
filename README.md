# RunForth

A (not so) simple online Forth runner.

## Introduction

RunForth provides a way to edit Forth code and execute it directly in the browser.

It provides a simple means of debugging by viewing stack contents on each executed line.

It also provides a means to accept keyboard input, though this requires some non-standard extensions.

You can [test it out here](https://rlamorea.github.io/runforth/).

## Caveats

RunForth uses the [ACE]() editor with default highlighting for Forth, which is not _quite_ accurate 
for the (most of) [ANS Forth]() [CORE words]() and [Extended CORE words]() that the [WAForth]() engine runs to interpret the code.

WAForth only implements Forth words in ALL CAPS. To accomodate this, RunForth automatically converts all text outside quotes and parentheses to ALL CAPS before running it.
This means all variables and words you define should be considered case-insensitive.

## Controls

Across the top of RunForth are a set of controls:

### RUN button

Clicking the `RUN` button will execute the Forth code in the editor, showing the output in the output panel on the right.

### Code / Workspace Selectors

RunForth automatically saves the code in the editor when the `Code` selector is selected.

Clicking the `Workspace` selector switches to an empty workspace to try out things while still preserving your code. When you are ready to go back to coding, use the `Code` selector and your code will be restored. The Workspace is never saved.

### Verbose option

By default, RunForth executes Forth in "Verbose" mode. In this mode it echoes each 
line of code that is run, shown in <span style="display: inline-block; color:gold; background: black;">GOLD</span>. 
The output (if any) from the interpreted line shows in <span style="display: inline-block; color:lightgreen; background: black;">LIGHT GREEN</span>.

Forth prompts are shown in <span style="display: inline-block; color:white; background: black;">white</span>, 
and errors in <span style="display: inline-block; color: red; font-weight: bold; background:black;">RED</span>.

If the `Verbose` option is off, only the <span style="display: inline-block; color:lightgreen; background: black;">OUTPUT</span> is shown.

### Keep Stack option

By default, RunForth resets the Forth engine every time you `RUN`. This means all words in the Dictionary are cleared and the Stack is emptied.

If the `Keep Stack` option is on, this will not happen and words will be remembered and the Stack will stay intact.

### Debugging

If the `Debug` option is on, RunForth will include debugging output. For most lines this will simply show the state of the Stack:

<div style="margin: 5en 0; background:black; color: yellow">STACK ( <i>N</i> ) <i>n1</i> <i>n2</i> ... <i>nN</i> </div>

Where `N` is the size of the Stack and `n1, n2 ... nN` is the contents of the Stack, with "top of stack" to the right.

When running a word you have defined in your code, you can also see the debugging of the lines _inside_ that word when it is executed. In this case the associated code line is also displayed, like:

<div style="margin: 5en 0; background:black; color: yellow">
two-plus-two .<br/>
<span style="color: darkgoldenrod; font-style: italic; font-size: 8pt;">5: : two-plus-two</span><br/>
<span style="color: yellow; font-size: 8pt;">STACK ( 0 )</span><br/>
<span style="color: darkgoldenrod; font-style: italic; font-size: 8pt;">5: 2 dup</span><br/>
<span style="color: yellow; font-size: 8pt;">STACK ( 2 ) 2 2</span><br/>
<span style="color: darkgoldenrod; font-style: italic; font-size: 8pt;">5: + ;</span><br/>
<span style="color: yellow; font-size: 8pt;">STACK ( 1 ) 4</span><br/>
<span style="color: lightgreen">4</span><br/>
<span style="color: yellow">STACK ( 0 )</span><br/>
<span style="color: white">ok</span></div>

## Directives

RunForth provides a few directives that are interpreted directly and not passed to Forth.

### `-rf-debug [<option>]`

The `-rf-debug` directive can be used to enable or disable debugging output.

It takes one optional parameter:

 - `on` - turn debugging output on for subsequent lines
 - `off` - turn debugging output off for subsequent lines
 - `toggle` - toggle debugging output for subsequent lines

No parameter or no recognizeable parameter defaults to `toggle`.

### `-rf-goto <line number>`

The `-rf-goto` directive will cause RunForth to continue executing at the specified line number (per the line numbering in the editor window).

### `-rf-goto-if <line number> [<match-value>]`

The `-rf-goto-if` directive will pop the value off the top of the Forth stack and check it.

 - If `match-value` is provided, a "true" result is when the stack value equals `match-value`
 - If no `match-value` is provided, any non-zero value is considered "true"

If the evaluation is "true", RunForth continues executing at the specified line number.

## Keyboard Input

WAForth provides an implemenation for the `KEY` word. This will bring up a "prompt" popup, take the result, and place all of the characters entered onto the Stack.

RunForth provides an additional word: `RFKEY`. When RFKEY is interpreted the output pane background will change slightly, and execution is stopped. When a key is pressed, the output background returns to black and execution resumes.

### Compiled Keyboard Input

The combination of how keyboard input is handled in browser and a lack of any real asynchronous support in WebAssembly (and therefore WAForth), if you place `KEY` or `RFKEY` inside a word, it just won't work. The input might be asked for, but the Forth code following that word will have long since executed.

This means that any use of `KEY` and `RFKEY` should be placed on its own line at the root level.

The `-rf-goto` and `-rf-goto-if` directives will let you build loops around keyboard input if necessary.

So if you had code like:

```forth
: wait-prompt ( -- )
    ." Press SPACE to Continue "
    0 0 do
       KEY 32 = if leave then
    loop
    ;
```

In RunForth you'd need to do this like:

```forth
: wait-prompt ( -- ) ." Press SPACE to Continue " ;
wait-prompt
RFKEY
32 =
-rf-goto-if 3
```

You could also do this with `-rf-goto-if` like:

```forth
: wait-prompt ( -- ) ." Press SPACE to Continue " ;
wait-prompt
RFKEY
-rf-goto-if 3 32
```

