---
layout: post
title: "Powershell: Everything you ever wanted to know about the switch statement"
date: 2018-01-12
tags: [PowerShell,Regex]
---

Like many other languages, PowerShell has commands for controlling the flow of executions within your scripts. One of those statement is the [switch](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_switch?view=powershell-5.1) statement and in PowerShell, it offers features that are not found in other languages. Today we will take a deep dive into working with the PowerShell `switch`.

<!--more-->

# Index

* TOC
{:toc}

# If statement

One of the first statements that you will learn is the `if` statement. It lets you execute a script block if a statement is true.

    if ( Test-Path $Path )
    {
        Remove-Item $Path
    }

You can have much more complicated logic by using `elseif` and `else` statements. Here is an example where I have a numeric value for day of the week and I want to get the name as a string.

    if ( $day -eq 0 ) { $result = 'Sunday' }
    elseif ( $day -eq 1 ) { $result = 'Monday' }
    elseif ( $day -eq 2 ) { $result = 'Tuesday' }
    elseif ( $day -eq 3 ) { $result = 'Wednesday' }
    elseif ( $day -eq 4 ) { $result = 'Thursday' }
    elseif ( $day -eq 5 ) { $result = 'Friday' }
    elseif ( $day -eq 6 ) { $result = 'Saturday' }

It turns out that this is a very common pattern and there are a lot of ways to deal with this. One of them is with a `switch`.

# Switch statement

The switch statement allows you to provide a variable and then provide a list of possible values. If the value matches the variable, then it's script will be executed.

    switch ( $day )
    {
        0 { $result = 'Sunday' }
        1 { $result = 'Monday' }
        2 { $result = 'Tuesday' }
        3 { $result = 'Wednesday' }
        4 { $result = 'Thursday' }
        5 { $result = 'Friday' }
        6 { $result = 'Saturday' }
    }

For this example, the value of `$day` matches one of the numeric values, then the correct name will be assigned to `$result`. We are only doing a variable assignment in this example, but any PowerShell can be executed in those script blocks.

## Assign to a variable

We can write that last example in another way.

    $result = switch ( $day )
    {
        0 { 'Sunday' }
        1 { 'Monday' }
        2 { 'Tuesday' }
        3 { 'Wednesday' }
        4 { 'Thursday' }
        5 { 'Friday' }
        6 { 'Saturday' }
    }

In this case, we are placing the value on the PowerShell pipeline and assigning it to the `$result`. You can do this same thing with `if` and `foreach` statements too.

## default

We can use the `default` keyword to identify the what should happen if there is no match.

    $result = switch ( $day )
    {
        0 { 'Sunday' }
        # ...
        6 { 'Saturday' }
        default { 'Unknown' }
    }

Here we are returning the value `Unknown` in the default case.

# Arrays

One of the cool features of the PowerShell `switch` is the way it handles arrays. If you give a `switch` an array, it will process each element in the array.

    $roles = @('WEB','Database')
    switch ( $roles ) {
        'Database'   { 'Configure SQL' }
        'WEB'        { 'Configure IIS' }
        'FileServer' { 'Configure Share' }
    }

If you have repeated items in your array, then they will be matched multiple times by the appropriate section.

## PSItem

You can use the `$PSItem` or `$_` to reference the current item that was processed. When we do a simple match, the value will be the value that we are matching. But I will be showing you some advanced matches in the next section where this will be used.

# Parameters

A unique feature of the PowerShell `switch` is that it has a number of switch parameters that change how it performs it matches.

## CaseSensitive

The matches are not case sensitive by default. If you need to be case sensitive then you can use `-CaseSensitive`. This can be used in combination with the other switch parameters.

## Wildcard

We can enable wildcard support with the `-wildcard` switch. This uses the same wildcard logic as the `-like` operator to do each match.

    switch -Wildcard ( $message )
    {
        'Error*'
        {
            Write-Error -Message $Message
        }
        'Warning*'
        {
            Write-Warning -Message $Message
        }
        default
        {
            Write-Output $message
        }
    }

Here we are processing a message and then outputting it on different streams based on the contents.

## Regex

The switch statement supports regex matches just like it does wildcards.

    switch -Wildcard ( $message )
    {
        '^Error'
        {
            Write-Error -Message $Message
        }
        '^Warning'
        {
            Write-Warning -Message $Message
        }
        default
        {
            Write-Output $message
        }
    }

I have more examples of using regex in another article I wrote: [The many ways to use regex](/2017-07-31-Powershell-regex-regular-expression).

## File

A little known feature of the switch statement is that it can process a file with the `-File` parameter. You use `-file` with a path to a file instead of giving it a variable expression.

    switch -Wildcard -File $path
    {
        'Error*'
        {
            Write-Error -Message $PSItem
        }
        'Warning*'
        {
            Write-Warning -Message $PSItem
        }
        default
        {
            Write-Output $PSItem
        }
    }

It works just like processing an array. In this example, I combine it with wildcard matching and use of the `$PSItem`. This would process a log file and convert it to warning and error messages depending on the regex matches.

# Advanced details

Now that you are aware of all these documented features, we can use them in the context of more advanced processing.

## Expressions

The `switch` can be on an expression instead of a variable.

    switch ( ( Get-Service | Where status -eq 'running' ).name ) {...}

Whatever the expression evaluates to will be the value used for the match.

## Multiple matches

You may have already picked up on this, but a `switch` can match to multiple conditions. This is especially true when using `-wildcard` or `-regex` matches. Be aware that you can add the same condition multiple times and all of them will trigger.

    switch ( 'Word' )
    {
        'word' { 'lowercase word match' }
        'WORD' { 'uppercase word match' }
        'WORD' { 'uppercase word match' }
    }

All three of these statements will fire. This shows that every condition is checked (in order). This holds true for processing arrays where each item will check each condition.

## Continue

Normally, this is where I would introduce the `break` statement, but it is better that we learn how to use `continue` first. Just like with a `foreach` loop, `continue` will continue onto the next item in the collection or exit the `switch` if there are no more items. We can rewrite that last example with continue statements so that only one statement executes.

    switch ( 'Word' )
    {
        'word' {
            'lowercase word match'
            continue
        }
        'WORD' {
            'uppercase word match'
            continue
        }
        'WORD' {
            'uppercase word match'
            continue
        }
    }

Instead of matching all three items, the first one is matched and the switch continues to the next value. Because there are no values left to process, the switch exits. This next example is showing how a wildcard could match multiple items.

    switch -Wildcard -File $path
    {
        '*Error*'
        {
            Write-Error -Message $PSItem
            continue
        }
        '*Warning*'
        {
            Write-Warning -Message $PSItem
            continue
        }
        default
        {
            Write-Output $PSItem
        }
    }

Because a line in the input file could contain both the word `Error` and `Warning`, we only want the first one to execute and then continue processing the file.

## Break

A `break` statement will exit the switch. This is the same behavior that `continue` will present for single values. The big difference is when processing an array. `break` will stop all processing in the switch and `continue` will move onto the next item.

    switch -Wildcard -File $path
    {
        'Error*'
        {
            Write-Error -Message $PSItem
            break;
        }
        '*Error*'
        {
            Write-Warning -Message $PSItem
            continue
        }
        '*Warning*'
        {
            Write-Warning -Message $PSItem
            continue
        }
        default
        {
            Write-Output $PSItem
        }
    }

In this case, if we hit any lines that start with `Error` then we will get an error and the switch will stop. This is what that `break` statement is doing for us. If we find `Error` inside the string and not just at the beginning, we will write it as a warning. We will do the same thing for `Warning`. It is possible that a line could have both the word `Error` and `Warning`, but we only need one to process. This is what the `continue` statement is doing for us.

## Break labels

The `switch` statement supports `break/continue` labels just like `foreach`.

    :filelist foreach($path in $logs)
    {
        :logFile switch -Wildcard -File $path
        {
            'Error*'
            {
                Write-Error -Message $PSItem
                break filelist
            }
            'Warning*'
            {
                Write-Error -Message $PSItem
                break logFile
            }
            default
            {
                Write-Output $PSItem
            }
        }
    }

I personally don't like the use of break labels but I wanted to point them out because they are confusing if you have never seen them before. When you have multiple `switch` or `foreach` statements that are nested, you may want to break out of more than the inner most item. You can place a label on a `switch` that can be the target of your `break`.

## ScriptBlock

Up until now, we have only matched values. We can use a scriptblock to perform the evaluation for a match if needed.

    switch ( $age )
    {
        {$PSItem -le 18} {
            'child'
        }
        {$PSItem -gt 18} {
            'adult'
        }
    }

I personally don't like this one very much because it adds a lot of complexity and can make your `switch` hard to read. In most cases where you would use something like this it would be better to use `if` and `elseif` statements. I would consider using this if I already had a large switch in place and I needed 2 items to hit the same evaluation block.

One thing that I think helps with legibility is to place the scriptblock in parentheses.

    switch ( $age )
    {
        ({$PSItem -le 18})
        {
            'child'
        }
        ({$PSItem -gt 18})
        {
            'adult'
        }
    }

It still executes the same and give a better visual break when quickly looking at it.

## Regex $matches

We need to revisit regex to touch on something that is not immediately obvious. The use of regex populates the `$matches` variable. I do go into the use of `$matches` more when I talk about [The many ways to use regex](/2017-07-31-Powershell-regex-regular-expression). Here is a quick sample to show it in action with named matches.

    $message = 'my ssn is 123-23-3456 and credit card: 1234-5678-1234-5678'
    switch -regex ($message)
    {
        '(?<SSN>\d\d\d-\d\d-\d\d\d\d)'
        {
            Write-Warning "message contains a SSN: $($matches.SSN)"
        }
        '(?<CC>\d\d\d\d-\d\d\d\d-\d\d\d\d-\d\d\d\d)'
        {
            Write-Warning "message contains a credit card number: $($matches.CC)"
        }
        '(?<Phone>\d\d\d-\d\d\d-\d\d\d\d)'
        {
            Write-Warning "message contains a phone number: $($matches.Phone)"
        }
    }

    WARNING: message may contain a SSN: 123-23-3456
    WARNING: message may contain a credit card number: 1234-5678-1234-5678

## $null

You can match a `$null` value that does not have to be the default.

    switch ( $value )
    {
        $null
        {
            'Value is null'
        }
        default
        {
            'value is not null'
        }
    }

Same goes for an empty string.

switch ( '' )
    {
        ''
        {
            'Value is empty'
        }
        default
        {
            'value is a empty string'
        }
    }

# Hashtables

One of my most popular posts is the one I did on [everything you ever wanted to know about hashtables](/2016-11-06-powershell-hashtable-everything-you-wanted-to-know-about/). One of the example use-cases for a `hashtable` is to be a lookup table. That is an alternate approach to a common pattern that a `switch` statement is often addressing.

    $lookup = @{
        0 = 'Sunday'
        1 = 'Monday'
        2 = 'Tuesday'
        3 = 'Wednesday'
        4 = 'Thursday'
        5 = 'Friday'
        6 = 'Saturday'
    }
    $result = $lookup[$day]

If I am only using a `switch` as a lookup, I will quite often use a `hashtable` instead.

## Enum

PowerShell 5.0 introduced the `Enum` and it is also an option in this case.

    enum DayOfTheWeek {
        Sunday
        Monday
        Tuesday
        Wednesday
        Thursday
        Friday
        Saturday
    }

    $result = [DayOfTheWeek]$day

We could go all day looking at different ways to solve this problem. I just wanted to make sure you knew you had options.

# Final words

The switch statement is simple on the surface but it offers some advanced features that most people don't realize are available. Stringing those features together makes this into a really powerful feature when it is needed. I hope you learned something that you had not realized before.