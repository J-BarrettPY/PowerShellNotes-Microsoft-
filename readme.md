Notes from the Automate administrative tasks by using PowerShell by Microsoft.

# Get-Command

PowerShell uses verbs, such as Get, and nouns, such as Random, to create cmdlets. This is called the verb-noun naming standard.
Use flags to target either the verb or the noun in the command you want. The flag you specify expects a value that's a string.
You can add pattern-matching characters to that string to ensure you express that, for example, a flag's value should start or end with a certain string.

These examples show how to use flags to filter a command list:

- -Noun: The `-Noun` flag targets the part of the command name that's related to the noun. That is, it targets everything after the hyphen (`-`). Here's a typical search for a command name:

```powershell
Get-Command -Noun a-noun*
```
This command searches for all cmdlets whose noun part starts with a-noun.

- -Verb: The -Verb flag targets the part of the command name that's related to the verb. You can combine the -Noun flag and the -Verb flag to create an even more detailed search query and type. Here's an example:

```powershell
Get-Command -Verb Get -Noun a-noun*
```
Now you've narrowed the search to specify that the verb part needs to match Get, and the noun part needs to match a-noun.

By using one or possibly two parameters, you can quickly find the cmdlet you need.

----------------------------

Complete Example:

Run the command Get-Command with the flag -Noun. Specify File* to find anything related to files.
```powershell
Get-Command -Noun File*
```

Run Get-Command. Specify the flags -Verb and -Noun.

```powershell
Get-Command -Verb Get -Noun File*
```

--------------------------

# Get-Help

By using the built-in help system in PowerShell, you can find out more about a specific command. You use the `Get-Command` cmdlet to locate a command that you need. After you've located the command, you might want to know more about what the command does and various ways to call it.

You can use the `Get-Help` core cmdlet to learn more about a command. Typically, you invoke `Get-Help` by specifying it by name and adding the `-Name` flag that contains the name of the cmdlet you want to learn about. Here's an example:
```powershell
Get-Help -Name Get-Help
```

If you don't want to display the full help page, narrow the response by adding flags to your Get-Help command. Here are some flags you can use:

- Full: Returns a detailed help page. It specifies information like parameters, inputs, and outputs that you don't get in the standard response.
- Detailed: Returns a response that looks like the standard response, but it includes a section for parameters.
- Examples: Returns only examples, if any exist.
- Online: Opens a web page for your command.
- Parameter: Requires a parameter name as an argument. It lists a specific parameter's properties.

For example, you can use the following command to return only the Examples section of the help page.

```powershell
Get-Help Get-FileHash -Examples
```

Because this output is difficult to read, you decide to use an alternative that is less verbose. That is, you use the `help` alias.

Enter the `help` command:
```powershell
help Get-FileHash
```

## Discover objects

When a cmdlet runs, it returns an object. When you invoke a cmdlet, the response you see has been formatted and might not necessarily represent all the available information for the response. To know more about what's being returned and how you can modify what is returned, you can use the command `Get-Member`.

The `Get-Member` cmdlet is meant to be piped on top of the command you run so that you can filter the output. A typical command-line invocation of `Get-Member` might look like the following example:

```powershell
Get-Process -Name 'name-of-process' | Get-Member
```

This command first produces an object result by calling `Get-Process`. That result is passed as an input to `Get-Member` by using the pipe (`|`). In return, you get a table result that includes the `Name`, `MemberType`, and `Definition` columns. You also get the type of the returned object.


## Filter a Get-Member result by using Select-Object

When you run `Get-Member`, the result is verbose. That is, many rows are returned. The object might have properties, like events and methods. To make the answer less verbose, you can filter on specific columns and also decide which columns to display. Keep in mind that the returned answer is already a subset of all the columns in the response.

Take a look at a `Get-Member` response that includes many columns. By introducing the `Select-Object` cmdlet, you can choose which columns appear in the response. The command expects either a comma-separated list of column names or a wildcard character, such as an asterisk (`*`), which indicates all columns.

When you use the `Select-Object` command in the context of `Select-Object Name, MemberType`, you specify only the columns you want. In this case, the columns are `Name` and `MemberType`. The command line would look like this:

```powershell
Get-Process -Name 'name-of-process' | Get-Member | Select-Object Name, MemberType
```

-------------------------

# Connect commands into a pipeline

Running a command can be powerful, you get data from your local machine or from across the network. To be even more effective, you need to learn how to get the data that you want. Most commands operate on objects, as input or as output, or both. Objects have properties and you may want to access a subset of those properties and present them in a report. You might also want to sort the data based on one or more properties. But how do you get there?

## Use Get-Member to inspect output

When you pass the results of a command to `Get-Member`, `Get-Member` returns information about an object, like:

- The type of object being passed to Get-Member.
- The Properties of the object that may be evaluated.
- The Methods of the object that may be executed.

Let's demonstrate this fact by running `Get-Member` on the command `Get-Process`.

```powershell
Get-Process | Get-Member
```

The output shows the type of object that the Get-Process command returns (System.Diagnostics.Process). The rest of the response shows the name, type, and definition of the object's members. You can see that If you want to fit Get-Process with another command in a pipeline, pairing it with Get-Member is a good first step.


## Select-Object

By default, when you run a command that is going to output to the screen, PowerShell automatically adds the command `Out-Default`. When the output data is a collection of objects, PowerShell looks at the object type to determine if there's a registered view for that object type. If it finds one, it uses that view.

The view generally doesn't contain all the properties of an object because it wouldn't display properly on screen, so only some of the most common properties are included in the view.

You can override the default view by using `Select-Object` and choosing your own list of properties. You can then send those properties to `Format-Table` or `Format-List`, to display the table however you like.

Consider the result of running `Get-Process` on the process `zsh`.

What you see is a view that represents what you most likely want to see from this command. However, this view doesn't show you a complete set of information. In order to see something different, you can explicitly specify which properties you want to see in the result.

## Getting the full response
What you've seen so far is a limited response. To present the full response, you use a wildcard `*`, like so: 

```powershell
Get-Process zsh | Format-List -Property *
```

The * character shows you every attribute and its value, which allows you to investigate the values you're interested in. The full response also uses presentation names for properties instead of the actual property names, and presentation names look good in a report.

## Selecting specific columns
To limit the response and find a middle ground between the default response and the full response, you want to select some properties you're interested in and have that as parameter input to `Select-Object`. But, and here's a problem, you need to use the real names for the columns. How do you find out the real names? Use `Get-Member`. A call to `Get-Member` gives you all the properties and their actual names.

From the default response, the properties `Id` and `ProcessName` are most likely called the same, but CPU(s) is a presentation name, real property names tend to consist of only text characters and no spaces. To find out the real name for a specific property, you can use `Get-Member`:

```powershell
Get-Process zsh | Get-Member -Name C*
```

You now get a list of all members with names that start with a `C`. Among them is `CPU`, which is likely what you want.

You now know how to use `Select-Object` to ask for exactly what you need with the correct property names, like so:

```powershell
Get-Process zsh | Select-Object -Property Id, Name, CPU
```

## Sorting
When you use `Sort-Object` in a pipeline, PowerShell sorts the output data by using the default properties first. If no such properties exist, it then tries to compare the objects themselves. The sorting is either by ascending or descending order.

By providing properties, you can choose to sort by specific columns, like so:

```powershell
Get-Process | Sort-Object -Descending -Property Name
```

In the preceding command, we're sorting by the column `Name` in descending order. To sort by more than one column, separate the column names with a comma, like so:

```powershell
Get-Process | Sort-Object -Descending -Property Name, CPU
```

In addition to sorting by column name, you can also provide your own custom expression. In this example, we use a custom expression to sort by the columns `Name` and `CPU` and control the sort order for each column.

```powershell
Get-Process 'some process' | Sort-Object -Property @{Expression = "Name"; Descending = $True}, @{Expression = "CPU"; Descending = $False}
```

The preceding example demonstrates how powerful and flexible `Sort-Object` can be. This subject is a bit advanced and out of scope for this module, but is revisited in more advanced modules.

--------------------------

## Discover the most-used processes on your machine

To manage your machine, you sometimes need to discover what processes run on it and how much memory and CPU they consume. This information tells you what the machine spends its resources on. You can use this information to decide whether to introduce new processes on your machine, to leave the machine as it is, or to free resources by closing resource-intensive processes. The more you know about the processes that run on your machine, the better.

- Type pwsh in a terminal window to start a PowerShell session:
```powershell
pwsh
```

- To begin, run the command `Get-Process`, and pipe in the cmdlets `Where-Object`, `Sort-Object`, and `Select-Object`.

```powershell
Get-Process | Where-Object CPU -gt 2 | Sort-Object CPU -Descending | Select-Object -First 3
```

The exact output you see depends on your machine. But, you should see a result where the first 3 (`-First 3`) processes whose CPU value is greater than 2 (`-gt 2`) are sorted in `-Descending` order.

--------------------------

## Use formatting and filtering

When you work with PowerShell, filtering and formatting are important concepts to understand, for a couple of reasons. First, you want to create a pipeline that produces the result you want. Second, you want to do so efficiently. Both in terms of how you pull data over the network, and how you ensure that the result is something you can work with.

## Filtering left
In a pipeline statement, filtering left means filtering for the results you want as early as possible. You can **think of the term left as early**, because PowerShell statements run from left to right. The idea is to make the statement fast and efficient by ensuring that the dataset you operate on is as small as possible. This principle really comes into play when your commands are operating on larger data stores or you're bringing back results across the network.

Consider the following statement:

```powershell
Get-Process | Select-Object Name | Where-Object Name -eq 'name-of-process'
```

This statement first retrieves all of the processes on the machine. It ends up formatting the response so that only the Name property is listed. This statement doesn't follow the filtering left principle, because it operates on all the processes, attempts to format the response, and then filters at the end.

It's better to filter first and then format, as in the following statement.

```powershell
Get-Process | Where-Object Name -eq 'name-of-process' | Select-Object Name
```

Often, a cmdlet that offers filtering is more efficient than using `Where-Object`. Here's a more efficient version of the preceding statement:

```powershell
Get-Process -Name 'name-of-process' | Select-Object Name
```

## Formatting right, formatting as the last thing you do

Whereas filtering left means to filter something as early as possible in a statement, formatting right means to format something as late as possible in the statement. Ok, but why do I need to format late? The answer is because format commands alter the structure of the object that your results are contained in so that your data is no longer found in the same properties. This alteration impacts your ability to retrieve the information you want by using pipe commands, `Select-Object`, or by looping through the results with `foreach`.

The formatting destroys the object with which you're dealing. Take the following call for example:

```powershell
Get-Process 'some process' | Select-Object Name, CPU | Get-Member
```

The type you get back is `System.Diagnostics.Process`. Now, add the `Format-Table` formatter like so:
```powershell
Get-Process 'some process' | Format-Table Name,CPU | Get-Member
```

----------------------------

# Write your first lines of code

## Step 1: Type the code in Cloud Shell

Azure Cloud Shell provides an in-browser experience to support our tutorial approach. Cloud Shell is at the right side of the webpage. It behaves like a normal PowerShell terminal window, but in a sandbox environment. You can type commands directly in the window, or you can run scripts you've already written and get the results in Cloud Shell.

In this module, you'll use a version of a code editor in Cloud Shell to write and run scripts.

In the Cloud Shell terminal, type the following code:

```powershell
New-Item HelloWorld.ps1
code HelloWorld.ps1
```

The `New-Item` command creates a new `.ps1` file in the current directory. The `.ps1` filename extension is the extension that's used for PowerShell scripts.

The `code` command followed by the filename of the script you want to work with opens the file in the Cloud Shell code editor. Another window opens where you can write and edit scripts and then save them to run in Cloud Shell. If you want to open a file that's stored in another location, you can define the full path instead of using only the filename.

In the code editor window, type the following code:

```powershell
Write-Output 'Hello World!'
```

## Step 2: Run the script
To run the script, enter the following command in the Cloud Shell terminal:

```powershell
. ./HelloWorld.ps1
```
*Be sure to include the dot (.) at the beginning of the command. This tells PowerShell to run the script or file that's being called.*

## Step 3: Observe the result
You should see the following output in Cloud Shell:

```
Hello World!
```


## Step 4: Create a new file and write code to receive input
In the open `HelloWorld.ps1` file, comment out the code you wrote in the editor by adding a number sign (`#`) before the command. Below the commented line, add the following lines of code:

```powershell
# Write-Output 'Hello World!'

$name = Read-Host -Prompt "Please enter your name"
Write-Output "Congratulations $name! You have written your first code with PowerShell!"
```

## How did your program work?
In this exercise, you invoked a cmdlet called `Write-Output`. Cmdlets are the main way you use PowerShell. The command syntax is a `Verb-Noun` format. This makes it easier to understand what the code is trying to do. The cmdlet's name is its intent. The code is doing something (verb) to a thing (noun).

`Hello World!` and the congratulatory sentence are both string inputs for the `Write-Output` cmdlet to process and output. A string is a basic data type that computers use. In PowerShell, you can enclose strings in either single quotation marks (`''`) or double quotation marks (`""`). For our code, we'll use double quotation marks to allow PowerShell to display variable values instead of variable names. You'll learn more about data types and how to define them in a later module.

By using `Read-Host`, you can write a message to prompt a user for input. You define the message for the user with the `-Prompt` parameter. Parameters allow a cmdlet to take input from a user. You store the input in a variable called `$name`, and then you use the `Write-Output` cmdlet to display the custom message in the Cloud Shell terminal.

## Recap
Let's take a moment to recap what you've learned in this first unit:

- Cmdlets are the main way to interact with PowerShell. They're written in a Verb-Noun format.
- Parameters take input so that a cmdlet can provide output or take action.
- PowerShell is a relaxed language. That is, it's case insensitive by default.
- PowerShell errors can help you identify issues, and reading errors carefully can save you time.
- Variables are used to store values that you want to use dynamically in your programs.

-------------------------

# What is a program?
A program is a set of instructions that complete computing tasks. The instructions are compiled into a format the computer can understand and then run by a user. A user can be a person or another program. The computer executes the instructions in order, one line at a time, until there are no more lines to execute or the program is explicitly told to stop.

Even the most basic programs do one or more of the following tasks:

- Accepts input from a source. Input includes information that:
  - Comes from a user who's typing on a keyboard or selecting controls on an interface.
  - Is retrieved from a file.
  - Is called from another program or network connection.
- Processes information, which includes:
  - Performing logic.
  - Performing mathematical calculations.
  - Manipulating data input to produce new data.
- Outputs results, which include information that's:
  - Displayed on a screen to a user.
  - Saved to a file.
  - Sent to another program.

A program can take different forms for different purposes. A program can be:

- A standalone application, such as a game, a text editor, billing software, and so on.
- A script, such as an advanced macro that executes inside another program to automate certain functionality.
- A combination of live code, equations, and data visualizations.

Some programs, including the examples in this module, need only a few lines of code. But complex programs such as operating systems need tens of thousands or sometimes millions of lines of code.

# What is syntax?
Like any spoken or written language, programming languages have their own grammatical rules, known as syntax. The syntax of any programming language includes keywords, operators, or other types of rules that might be specific to that language.

Keywords are specific words reserved by a programming language that have special meaning and behavior. In PowerShell, many of the keywords read like English. For example, `if`, `while`, and `return` are keywords you can use to write code in PowerShell and many other languages.

Operators are special characters, such as parentheses (`()`) or equal signs (`=`). These characters tell the computer to perform specific mathematical, relational, or logical operations to produce a result.

When you typed your code in the Cloud Shell terminal in the preceding unit, you might have noticed small changes to the color of the text and symbols. This color coding is called syntax highlighting. As you're reading your code, syntax highlighting can help you spot any mistakes you might have made. This feature is available and even more robust in many code editors, such as Visual Studio Code.

## Compiling code in PowerShell
Computers aren't good at reading our programs in the way we write them. Programming languages need to be translated into a form that the computer can understand. Programming languages have various ways of doing this.

Many programming languages compile code as an individual step. You write your code, run it through a special program called a compiler, and the compiler produces an executable package to run.

Other languages, such as Python, have what's called an interpreter, which interprets the code for the computer and executes the code one line at a time as it's interpreting it.

PowerShell works both a little differently and a little similarly to the compiled and interpreted approaches.

PowerShell is compiled into an abstract syntax tree (AST), first in memory, and then run. But you don't need to do a deep dive here to use PowerShell. All you need to know is that the computer checks your code first in the AST as it looks for major issues. Then, if everything is okay, your program is run by the computer without the need for a compiled executable program. This approach is useful, because it ensures that your code runs correctly before it's run by the computer, where it might otherwise make changes and stop because of a syntax error. By contrast, an interpreted language such as Python runs the code until it finds something wrong in the syntax.


## Exploring PowerShell
An important feature of PowerShell is its built-in help system, which gives you quick access to information about PowerShell commands. If you get stuck as you're writing, you can look up help for commands or PowerShell concepts by using the `Get-Help` command. For example, to see all the details about the `Write-Output` command, you can type and run the following command:

```powershell
Get-Help -Name 'Write-Output' -Full
```

`Get-Help` is the command to run and `Write-Output` is the name of the command to get help for. The `-Full` switch tells PowerShell to get all information for the specified command, including a command description, parameter information, examples, and more. This help information is accessible in any PowerShell terminal, including the Azure Cloud Shell terminal.

If you want to explore all the commands that PowerShell has to offer, you can use `Get-Command *` to view the full list. The asterisk (`*`) is a wildcard character in PowerShell. It allows you to match patterns to find information more dynamically. In this case, you use the `*` to filter for all available commands. For example, to get all commands that have `User` in them, run `Get-Command *User*`.

Another great thing about PowerShell is that it comes with an integrated shell. By using the shell, you can test your code and interact with the output without having to run your code every time you want to test something. To ensure that your code works as expected, you can type it right in the terminal.


# Introduction to scripting

PowerShell scripting is the process of writing a set of statements in the PowerShell language and storing those statements in a text file. Why would you do that? After you use PowerShell for a while, you find yourself repeating certain tasks, like producing log reports or managing users. When you've repeated something frequently, it's probably a good idea to automate it: to store it in such a way that makes it easy to reuse.

The steps to automate your task usually include calls to cmdlets, functions, variables, and more. To store these steps, you'll create a file that ends in .ps1 and save it. You'll then have a script you can run.

Before you start learning to script, let's get an overview of the features of the PowerShell scripting language:

- **Variables**. You can use variables to store values. You can also use variables as arguments to commands.

- **Functions**. A function is a named list of statements. Functions produce an output that display in the console. You can also use functions as input for other commands.

- **Flow control**. Flow control is how you control various execution paths by using constructs like If, ElseIf, and Else.

- **Loops**. Loops are constructs that let you operate on arrays, inspect each item, and do some kind of operation on each item. But loops are about more than array iteration. You can also conditionally continue to run a loop by using Do-While loops. For more information, see About Do.

- **Error handling**. It's important to write scripts that are robust and can handle various types of errors. You'll need to know the difference between terminating and non-terminating errors. You'll use constructs like Try and Catch. We'll cover this topic in the last conceptual unit of this module.

- **Expressions**. You'll frequently use expressions in PowerShell scripts. For example, to create custom columns or custom sort expressions. Expressions are representations of values in PowerShell syntax.

- **.NET** and **.NET Core integration**. PowerShell provides powerful integration with .NET and .NET Core. This integration is beyond the scope of this module.

## Run a script
You need to be aware that some scripts aren't safe. If you find a script on the internet, you probably shouldn't run it on your computer unless you understand exactly what it does. Even with scripts you consider safe, there might be a risk. For example, imagine a script that cleans things up in a test environment. That script might be harmful in a production environment. You need to understand what a script does, whether it was written by you or by a colleague or if you got it from the internet.

PowerShell attempts to protect you from doing things unintentionally in two main ways:

- **Requirement to run scripts by using a full path or relative path**. When you run a script, you always need to provide the script's path. Providing the path helps you to know exactly what you're running. For example, there could be commands and aliases on your computer you don't intend to run, but that have the same name as your script. Including the path provides an extra check to ensure you run exactly what you want to run.
- **Execution policy**. An execution policy is a safety feature. Like requiring the path of a script, a policy can stop you from doing unintentional things. You can set the policy on various levels, like the local computer, current user, or particular session. You can also use a Group Policy setting to set execution policies for computers and users.

These two mechanisms don't stop you from opening a file, copying its contents, placing the contents in a text file, and running the file. They also don't stop you from running the code via the console. These mechanisms help to stop you from doing something unintentional. They aren't a security system.

To create and run a script:

- Create some PowerShell statements like the following and save them in a file that ends with .ps1:

```powershell
# PI.ps1
$PI = 3.14
Write-Host "The value of `$PI is $PI"
```

- Run the script by invoking it by its name and path:
```powershell
./PI.ps1
```

## Execution policy
You can manage execution policy using these cmdlets:

- `Get-ExecutionPolicy`. This cmdlet returns the current execution policy. On Linux and macOS, the value returned is `Unrestricted`. For these operating systems, you can't change the value. That limitation doesn't make Linux or Mac any less safe. Remember, an execution policy is a safety feature, not a security mechanism.

- `Set-ExecutionPolicy`. If you're using a Windows computer, you can use this cmdlet to change the value of an execution policy. It takes an `-ExecutionPolicy` parameter. There are a few possible values. It's a good idea to use `Default` as the value. That value sets the policy to `Restricted` on Windows clients and `RemoteSigned` on Windows Server. `Restricted` means you can't run scripts. You can run only commands, which makes sense on a client. `RemoteSigned` means that scripts written on the local computer can run. Scripts downloaded from the internet need to be signed by a digital signature from a trusted publisher.

## Variables
Variables aren't just for scripts. You can also define them on the console. You can store values in variables so you can use them later. To define a variable, precede it with the $ character. Here's an example:

```powershell
$PI = 3.14
```


## Working with variables: Quotation marks and interpolation
When you output text via `Write-Host` or `Write-Output`, you can use single or double quotation marks. Your choice depends on whether you want to interpolate the values. There are three mechanisms you should know about:

- **Single quotation marks**. Single quotation marks specify literals; what you write is what you get. Here's an example:

```powershell
Write-Host 'Here is $PI' # Prints Here is $PI
```

If you want to interpolate — to get the value of `$PI` interpreted and printed — you need to use double quotation marks.

- **Double quotation marks**. When you use double quotation marks, variables in strings are interpolated:

```powershell
Write-Host "Here is `$PI and its value is $PI" # Prints Here is $PI and its value is 3.14
```

There are two things going on here. The back tick (`) lets you escape what would be an interpolation of the first instance of `$PI`. In the second instance, the value is interpolated and is written out.

- `$()`. You can also write an expression within double quotation marks. To do that, use the `$()` construct. One way to use this construct is to interpolate properties of objects. Here's an example:

```powershell
Write-Host "An expression $($PI + 1)" # Prints An expression 4.14
```

## Scope
Scope is how PowerShell defines where constructs like variables, aliases, and functions can be read and changed. When you're learning to write scripts, you need to know what you have access to, what you can change, and where you can change it. If you don't understand how scope works, your code might not work as you expect it to.

## Types of scope
Let's talk about the various scopes:

- **Global scope**. When you create constructs like variables in this scope, they continue to exist after your session ends. Anything that's present when you start a new PowerShell session can be said to be in this scope.

- **Script scope**. When you run a script file, a script scope is created. For example, a variable or a function defined in the file is in the script scope. It will no longer exist after the file is finished running. For example, you can create a variable in the script file and target the global scope. But you need to explicitly define that scope by prepending the variable with the `global` keyword.

- **Local scope**. The local scope is the current scope, and can be the global scope or any other scope.


## Scope rules
Scope rules help you understand what values are visible at a given point. They also help you understand how to change a value.

- **Scopes can nest**. A scope can have a parent scope. A parent scope is an outer scope, outside of the scope you're in. For example, a local scope can have the global scope as a parent scope. Conversely, a scope can have a nested scope, also known as a child scope.

- **Items are visible in the current and child scopes**. An item, like a variable or a function, is visible in the scope in which it's created. By default, it's also visible in any child scopes. You can change that behavior by making the item private within the scope. Here's an example that uses a variable defined in the console:

```powershell
$test = 'hi'
```

If you have a Script.ps1 file that contains the following content, it will print "hi" when the script runs:

```powershell
Write-Host $test # Prints hi
```

You can see that the variable $test is visible in both the local scope and its child scope, in this case, the script scope.

- **Items can be changed only in the created scope**. By default, you can change an item only in the scope in which it was created. You can change this behavior by explicitly specifying a different scope.

## Profiles

A profile is a script that runs when PowerShell starts. You can use a profile to customize your environment to, for example, change background colors and errors and do other types of customizations. PowerShell will apply these changes to each new session you start.

## Profile types
PowerShell supports several profile files. You can apply them at various levels, as you see here:

| Description                  | Path                                         |
|------------------------------|----------------------------------------------|
| All users, all hosts         | $PSHOME\Profile.ps1                         |
| All users, current host      | $PSHOME\Microsoft.PowerShell_profile.ps1   |
| Current user, all hosts      | $Home[My ]Documents\PowerShell\Profile.ps1 |
| Current user, current host   | $Home[My ]Documents\PowerShell\Microsoft.PowerShell_profile.ps1 |

There are two variables here: `$PSHOME` and `$Home`. `$PSHOME` points to the installation directory for PowerShell. `$Home` is the current user's home directory.

## Create a profile
When you first install PowerShell, there are no profiles, but there is a `$Profile` variable. It's an object that points to the path where each profile to apply should be placed. To create a profile:

- Decide the level on which you want to create the profile. You can run `$Profile | Select-Object *` to see the profile types and the paths associated with them.

- Select a profile type and create a text file at its location by using a command like this one: `New-Item -Path $Profile.CurrentUserCurrentHost`.

- Add your customizations to the text file and save it. The next time you start a session, your changes will be applied.

## Parameters

After you've created a few scripts, you might notice your scripts aren't flexible. Going into your scripts to change them isn't efficient. There's a better way to handle changes: use parameters.

Using parameters makes your scripts flexible, because it allows users to select options or send input to the scripts. You won't need to change your scripts as frequently, because in some cases you'll just need to change a parameter value.

Cmdlets, functions, and scripts all accept parameters.

## Declare and use a parameter
To declare a parameter, you need to use the keyword `Param` with an open and close parenthesis:

```powershell
Param()
```

Inside the parentheses, you define your parameters, separating them with commas. A typical parameter declaration might look like this:

```powershell
# CreateFile.ps1
Param (
  $Path
)
New-Item $Path # Creates a new file at $Path.
Write-Host "File $Path was created"
```
The script has a `$Path` parameter that's later used in the script to create a file. The script is now more flexible.

## Use the parameter
To call a script with a parameter, you need to provide a name and a value. Assume the above script is called `CreateFile.ps1`. You could call it like this:

```powershell
./CreateFile.ps1 -Path './newfile.txt' # File ./newfile.txt was created.
./CreateFile.ps1 -Path './anotherfile.txt' # File ./anotherfile.txt was created.
```

Because you used a parameter, you don't need to change the script file when you want to call the file something else.

## Improve your parameters
When you first create a script that uses parameters, you might remember exactly what the parameters are for and what values are reasonable for them. As time passes, you might forget those details. You might also want to give a script to a colleague. The solution in these cases is to be explicit, which makes your scripts easy to use. You want a script to fail early if it passes unreasonable parameter values. Here are some things to consider when you define parameters:

- **Is it mandatory**? Is the parameter optional or required?
- **What values are allowed**? What values are reasonable?
- **Does it accept more than one type of value**? Does the parameter accept any type of value, like string, Boolean, integer, and object?
- **Can the parameter rely on a default**? Can you omit the value altogether and rely on a default value instead?
- **Can you further improve the user experience**? Can you be even clearer to your user by providing a Help message?

## Select an approach
All parameters are optional by default. That default might work in some cases, but sometimes you need your user to provide parameter values, and the values need to be reasonable ones. If the user doesn't provide a value to a parameter, the script should quit or tell the user how to fix the problem. The worst scenario is for the script to continue and do things you don't want it to do.

There are a couple of approaches you can use to make your script safer. You can write custom code to inspect the parameter value. Or, you can use decorators that do roughly the same thing. Let's look at both approaches.

- Use `If/Else`. The `If/Else` construct allows you to check the value of a parameter and then decide what to do. Here's an example:

```powershell
Param(
   $Path
)
If (-Not $Path -eq '') {
   New-Item $Path
   Write-Host "File created at path $Path"
} Else {
   Write-Error "Path cannot be empty"
}
```

The script will run `Write-Error` if you don't provide a value for `$Path`.

- Use the `Parameter[]` decorator. A better way, which requires less typing, is to use the `Parameter[]` decorator:

```powershell
Param(
   [Parameter(Mandatory)]
   $Path
)
New-Item $Path
Write-Host "File created at path $Path"
```

If you run this script and omit a value for `$Path`, you end up in a dialog that prompts for the value:

```
cmdlet CreateFile.ps1 at command pipeline position 1
Supply values for the following parameters:
Path:
```

You can improve this decorator by providing a Help message users will see when they run the script:

```powershell
[Parameter(Mandatory, HelpMessage = "Please provide a valid path")]
```

When you run the script, you get a message that tells you to type `!?` for more information:

```powershell
cmdlet CreateFile.ps1 at command pipeline position 1
Supply values for the following parameters:
(Type !? for Help.)
Path: !?  # You type !?
Please provide a valid path  # Your Help message.
```

- **Assign a type**. If you assign a type to a parameter, you can say, for example, that the parameter accepts only strings, not Booleans. That way, the user knows what to expect. You can assign a type to a parameter by preceding it with the type enclosed in brackets:

```powershell
Param(
   [string]$Path
)
```

These three approaches aren't mutually exclusive. You can combine them to make your script safer.

-------------------

# Flow control

Flow control refers to how your code runs in your console or script. It describes the flow the code follows and how you control that flow. There are various constructs available to help you control the flow. The code can run all the statements, or only some of them. It can also repeat certain statements until it meets a certain condition.

Let's examine these flow-control constructs to see what they can do:

- **Sanitize input**. If you use parameters in a script, you need to ensure your parameters hold reasonable values so your script works as intended. Writing code to manage this process is called sanitizing input.

- **Control execution flow**. The previous technique ensures you get reasonable and correct input data. This technique is more about deciding how to run code. The values set can determine which group of statements runs.

- **Iterate over data**. Sometimes your data takes the form of an array, which is a data structure that contains many items. For such data, you might need to examine each item and perform an operation for each one. Many constructs in PowerShell can help you with that process.

## Manage input and execution flow by using If, ElseIf, and Else

You can use an `If` construct to determine if an expression is `True` or `False`. Depending on that determination, you might run the statement defined by the `If` construct. The syntax for `If` looks like this:

```powershell
If (<expression that evaluates to True or False>) 
{
  # Statement that runs only if the preceding expression is $True.
}
```

## Operators
PowerShell has two built-in parameters to determine if an expression is True or False:

- `$True` indicates that an expression is `True`.
- `$False` indicates that an expression is `False`.
You can use operators to determine if an expression is `True` or `False`. There are a few operators. The basic idea is usually to determine if something on the left side of the operator matches something on the right side, given the operator's condition. An operator can express conditions like whether something is equal to something else, larger than something else, or matches a regular expression.

Here's an example of using an operator. The `-le` operator determines if the value on the left side of the operator is less than or equal to the value on the right side:

```powershell
$Value = 3
If ($Value -le 0) 
{
  Write-Host "Is negative"
}
```

This code won't display anything because the expression evaluates to `False`. The value 3 is clearly positive.

## Else
The `If` construct runs statements only if they evaluate to `True`. What if you want to handle cases where they evaluate to `False`? That's when you use the `Else` construct. `If` expresses "if this specific case is true, run this statement." `Else` doesn't take an expression. It captures all cases where the `If` clause evaluates to `False`. When `If` and `Else` are combined, the code runs the statements in one of the two constructs. Let's modify the previous code to include an `Else` construct:

```powershell
$Value = 3
If ($Value -le 0) 
{
  Write-Host "Is negative"
} Else {
  Write-Host "Is Positive"
}
```

Because we put the `Else` next to the ending brace for the `If`, we created a joined construct that works as one. If you run this code in the console, you'll see that Is Positive prints. That's because `If` evaluates to `False`, but `Else` evaluates to `True`. So `Else` prints its statement.

## ElseIf
`If` and `Else` work great to cover all the paths code can take. `ElseIf` is another construct that can be helpful. `ElseIf` is meant to be used with `If`. It says "the expression in this construct will be evaluated if the preceding `If` statement evaluates to `False`." Like `If`, `ElseIf` can take an expression, so it helps to think of `ElseIf` as a secondary `If`.

Here's an example that uses `ElseIf`:

```powershell
# _FullyTax.ps1_
# Possible values: 'Minor', 'Adult', 'Senior Citizen'
$Status = 'Minor'
If ($Status -eq 'Minor') 
{
  Write-Host $False
} ElseIf ($Status -eq 'Adult') {
  Write-Host $True
} Else {
  Write-Host $False
}
```

It's possible to write this code in a more compact way, but this way does show the use of `ElseIf`. It shows how `If` is evaluated first, then `ElseIf`, and then `Else`.

-------------------

# Error handling

So far, you've seen how adding parameters and flow-control constructs can make your scripts flexible and safer to use. But sometimes, you'll get errors in your scripts. You need a way to handle those errors.

Here are some factors to consider:

- **How to handle the error**. Sometimes you get errors you can recover from, and sometimes it's better to stop the script. It's important to think about the kinds of errors that can happen and how to best manage them.

- **How severe the error is**. There are various kinds of error messages. Some are more like warnings to the user that something isn't OK. Some are more severe, and the user really needs to pay attention. Your error-handling approach depends on the type of error. The approach could be anything from presenting a message to raising the severity level and potentially stopping the script.

## Errors
A cmdlet or function, for example, might generate many types of errors. We recommend that you write code to manage each type of error that might occur, and that you manage them appropriately given the type. For example, say you're trying to write to a file. You might get various types of errors depending on what's wrong. If you're not allowed to write to the file, you might get one type of error. If the file doesn't exist, you might get another type of error, and so on.

There are two types of errors you can get when you run PowerShell:

- **Terminating error**. An error of this type will stop execution on the row where the error occurred. You can handle this kind of error using either `Try-Catch` or `Trap`. If the error isn't handled, the script will quit at that point and no statements will run.

- **Non-terminating error**. This type of error will notify the user that something is wrong, but the script will continue. You can upgrade this type of error to a terminating error.

## Managing errors by using Try/Catch/Finally
You can think of a terminating error as an unexpected error. These errors are severe. When you deal with one, you should consider what type of error it is and what to do about it.

There are three related constructs that can help you manage this type of error:

- `Try`. You'll use a `Try` block to encapsulate one or more statements. You'll place the code that you want to run — for example, code that writes to a data source — inside braces. A `Try` must have at least one `Catch` or `Finally` block. Here's how it looks:

```powershell
Try {
   # Statement. For example, call a command.
   # Another statement. For example, assign a variable.
}
```

- `Catch`. You'll use this keyword to catch or manage an error when it occurs. You'll then inspect the exception object to understand what type of error occurred, where it occurred, and whether the script can recover. A `Catch` follows immediately after a `Try`. You can include more than one `Catch` — one for each type of error — if you want. Here's an example:

```powershell
Try {
   # Do something with a file.
} Catch [System.IO.IOException] {
   Write-Host "Something went wrong"
}  Catch {
   # Catch all. It's not an IOException but something else.
}
```

The script tries to run a command that does some I/O work. The first Catch catches a specific type of error: [System.IO.IOException]. The last Catch catches anything that's not a [System.IO.IOException].

- `Finally`. The statements in this block will run regardless of whether anything goes wrong. You probably won't use this block much, but it can be useful for cleaning up resources, for example. To use it, add it as the last block:

```powershell
Try {
   # Do something with a file.
} Catch [System.IO.IOException] {
   Write-Host "Something went wrong"
}  Catch {
   # Catch all. It's not an IOException but something else.
} Finally {
   # Clean up resources.
}
```

## Inspecting errors
We've talked about exception objects in the context of catching errors. You can use these objects to inspect what went wrong and take appropriate measures. An exception object contains:

- **A message**. The message tells you in a few words what went wrong.

- **The stacktrace**. The stacktrace tells you which statements ran before the error. Imagine you have a call to function A, followed by B, followed by C. The script stops responding at C. The stacktrace will show that chain of calls.

- **The offending row**. The exception object also tells you which row the script was running when the error occurred. This information can help you debug your code.

So how do you inspect an exception object? There's a built-in variable, `$_`, that has an `exception` property. To get the error message, for example, you would use `$_.exception.message`. In code, it might look like this:

```powershell
Try {
     # Do something with a file.
   } Catch [System.IO.IOException] {
     Write-Host "Something IO went wrong: $($_.exception.message)"
   }  Catch {
     Write-Host "Something else went wrong: $($_.exception.message)"
   }
```

## Raising errors
In some situations, you might want to cause an error:

- **Non-terminating errors**. For this type of error, PowerShell just notifies you that something went wrong, by using the `Write-Error` cmdlet, for example. The script continues to run. That might not be the behavior you want. To raise the severity of the error, you can use a parameter like `-ErrorAction` to cause an error that can be caught with `Try/Catch`, like so:

```powershell
Try {
   Get-Content './file.txt' -ErrorAction Stop
} Catch {
   Write-Error "File can't be found"
}
```

By using the -ErrorAction parameter and the value Stop, you can cause an error that Try/Catch can catch.

- **Business rules**. You might have a situation where the code doesn't actually stop responding, but you want it to for business reasons. Imagine you're sanitizing input and you check whether a parameter is a path. A business requirement might specify only certain paths are allowed, or the path needs to look a certain way. If the checks fail, it makes sense to throw an error. In a situation like this, you can use a `Throw`.
