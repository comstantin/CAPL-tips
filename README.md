# Vector CAPL, Getting Started:  Everything you need to know but no one will tell you.

This page does not aim to provide a comprehensive introduction to *CAPL*, rather it provides information necessary to accomplish basic things with *CAPL* that is unnecessarily difficult to find elsewhere.  Some more general information can be found at the following links:

<https://www.vector.com/int/en/know-how/capl/#>  
<https://cdn.vector.com/cms/content/products/VectorCAST/Events/TechNights/CAPLQuickstart_Generic_2018_Final.pdf>  
<https://www.embeddedc.in/p/n-capl-can-accessprogramming-language-n.html>  
<https://robertscaplblog.files.wordpress.com/2016/02/capl_function_reference_manual.pdf>  <-- Defines built-in functions

## Initial Setup

There are a number of ways to use a *CAPL* script, and they each have, sometimes subtle, sometimes substantial differences in effect.  There are also differences between how *CAPL* scripts are used in *CANoe* and *CANalyzer*.  

When a *CAPL* script is inserted as a *Program Node* in the *Measurement Setup* window, it does not act at all like a node on the network. Instead, it acts as a transformer: traffic enters the node from the left, is wholly consumed by the script, and only *messages* (or *frames* etc.) output by the script flow downstream to the right.  For a *CAPL* script to act as a network node, it must be inserted into the loopback path of the *Measurement Setup* window in *CANalyzer*, and as a *Network Node* (not a *Test Module*) in the *Simulation Setup* window in *CANoe*.

It is best to always edit the *CAPL* script by right-clicking its node in *CANoe*/*CANalyzer*.  Otherwise, the database file will not be loaded.  The database file is not associated with the script.

## Output

The `output()` function will take a *message* (*frame* etc.) as its argument, and will send the *message* downstream, if configured as a *Program Node*, and out onto the network if configured as a *Network Node*.

## Write

The `write()` function prints to the *Write* window as a form of console output.  It is similar to `printf()` in c and takes the same arguments.  There are, however, a few critical differences: it always inserts a newline (look at `writeEx()` to avoid this) and it ignores tab (`\t`) characters.  Similarly to `printf()`, if the format string does not match the given type, it will not be converted and nonsense will be displayed.  For example, `.phys` will always produce a `float`, even if the scale factor means it will always be an integer, and passed to `write()` with a format string of `%d`, will always produce 0.

Choosing the *CAPL/.NET* tab in the *Write* window removes unnecessary clutter from the output making the output suitable for input to a spreadsheet or another program. 

## Timestamp

The system does not automatically timestamp *Write* output unlike the *Trace* window.  To achieve similar functionality `timenow()/100000.0` can be used.  This provides one fewer digit of precision than the *Trace* window for whatever reason.

## Signal Values

The *CAPL* system can use the loaded database (`.dbc`, `.ldf`, etc.) to automatically interpret and set *signal* values, but nowhere is this concisely described.  The *signal* name itself is used to set or get the raw (on the wire) value of the *signal*.  To get or set the scaled value, append `.phys` to the *signal* variable name.  

The values defined in a *value table* (enumeration) associated with the *signal* can also be accessed.  To get a string representing the current value of the *signal*, use 
`((enum VtSig_<signal name>)<variable>).name()` (e.g. `((enum VtSig_ShifterPosition)this).name()`).  

The *value table* values can be used as if they were defined as constants, but if (as is often the case) multiple *signals* use the same values, the system is not smart enough to figure out which one is meant.  `VtSig_<signal name>::<value>` can be used in this case (e.g. `VtSig_ShifterPosition::Park` would return the numerical value of `Park` as defined in the *value table* associated with the `ShifterPosition` *signal*).  

**NOTE: `<signal name>::<value>` can be used in most situations instead, but breaks if the value name is unique, and some other situations for unclear reasons. The `VtSig_` notation is much more reliable.** 

Also Note: value tables may not use the `VtSig_<signal name>` nomenclature depending on the type of database and how it was created.  Replace in the above snippets with the value table name as defined in the database as necessary.

## Receiving data from the network

There are two ways to perform a task when new data is received from the network: `on message` (or `on frame` when using *LIN*), and `on signal`. `on message` is called every time a particular *message* is received (or any *message* not otherwise defined with `on message *`), and `on signal` is called only when the value in a particular *signal* changes.   

The this identifier is used in an `on` function to refer to whatever data (*message*, *frame*, *signal*, etc.) the function is called on.

## Tips

- Enable all compiler warnings in the options of CANoe and be sure to keep an eye out for them
- Include comments in your code. Comments play a crucial role in making your code more understandable.
- Use name prefixes to indicate the purpose or type of the variable or function, for example:
  - cName &nbsp; constants
  - gName &nbsp; global variables
  - utName &nbsp;utility functions
  - pName &nbsp; function parameters
- It's generally a good practice to avoid using complex calculations within control flow statements or conditions, as this can make your code more difficult to read and debug. Instead, consider assigning the intermediate results of any complex calculations to local variables. This not only improves the readability of your code but also makes it much easier to track the flow of data and isolate bugs.
- It's a good practice to always use curly brackets, even if they are not strictly necessary for single statements after an if statement. This helps to prevent logical errors and establishes a clear, consistent code structure for easier understanding and debugging.
- Event handlers should be designed to execute quickly, rather than running for an extended period of time.
- Design your code to enable reuse through the use of CAPL include files
- Assign return values to local variables for debugging visibility and ease of use.

### Test modules

- Implement functions to promote reusability in test cases, which can enhance code readability, simplify maintenance, and minimize development effort.
- Conduct thorough return value checks and document any issues encountered through the use of the TestStepFail() function, to promote efficient and effective testing and debugging.