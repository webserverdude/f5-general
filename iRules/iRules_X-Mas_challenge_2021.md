# iRules X-Mas challenge 2021

## The iRule
Look at this iRule, it is deliberately insecure. The cookie has no secure flag and it's not encrypted.
The iRule is supposed to read the username value from the Cookie _X-User_ and store it into the variable _$cookie_. 
Then it will substitute the value of _$cookie_ with the string _Username has logged in_ and save it to the variable _$user_. 
This will then be logged to the /var/log/ltm.

```tcl
when HTTP_REQUEST {
    set cookie [HTTP::cookie value X-User]
    set user [subst "${cookie} has logged in"]
    log local0. "Logging: $user"
    return
}
```
This iRule would produce the following output to the LTM log:

`Dec 22 08:12:29 bigip.intern info tmm1[10955]: Rule /Common/ir_xmas_challenge <HTTP_REQUEST>: Logging: "DanielW has logged in"`

## The question
Why is this iRule insecure? What kind of vulnerability are you exposing yourself to by running this iRule?

## The answer
This iRule is vulnerable to code injection, it allows an attacker to inject Tcl commands.
Quoting from [K15650046: Tcl code injection security exposure](https://support.f5.com/csp/article/K15650046)

_"The design of the Tcl language allows for substitutions in statements and commands and this feature of Tcl can allow injection attacks similar to those seen in SQL or shell scripting languages, where arbitrary user input is interpreted as code and executed.</br> The best practice for Tcl scripting is to enclose all expressions, ensuring that they are not substituted or evaluated unexpectedly. An additional benefit of this practice is increased performance, as the expressions can be precompiled instead of re-evaluated dynamically at runtime."_

## The exploit

Replacing the cookie value with TCL `[TCP::respond [info level 0]]` will expose the iRule context in which the TCL was executed.

![Exploit](/assets/xmas-challenge-2021.png "Exploit")

## Solution
Use curly braces around _$cookie_ to ensure the expression is evaluated without substitution:

```tcl
when HTTP_REQUEST {
    set cookie [HTTP::cookie value X-User]
    set user [subst {"${cookie} has logged in"}]
    log local0. "Logging: $user"
    return
}
```