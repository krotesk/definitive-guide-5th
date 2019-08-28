---
description: Погружение в диалплан
---

# Глава 10

> For a list of all the ways technology has failed to improve the quality of life, please press three.
>
> Alice Kahn

Alrighty. You’ve got the basics of dialplans down, but you know there’s more to come. If you don’t have [Chapter 6](https://www.oreilly.com/library/view/asterisk-the-definitive/9781492031598/ch06.html#asterisk-DP-Basics) sorted out yet, please go back and give it another read. We’re about to get into more advanced topics.

## Expressions and Variable Manipulation

As we begin our dive into the deeper aspects of dialplans, it is time to introduce you to a few tools that will greatly add to the power you can exercise in your dialplan. These constructs add incredible intelligence to your dialplan by enabling it to make decisions based on different criteria you define. Put on your thinking cap, and let’s get started.

{% hint style="info" %}
**Note**

Throughout this chapter we use best practices that have been developed over the years in dialplan creation. The primary one you’ll notice is that all the first priorities start with the `NoOp()` application \(which simply means No Operation; nothing functional will happen\). The other one is that all following lines will start with `same => n`, which is a shortcut that says, “Use the same extension as was just previously defined.” Additionally, the indentation is two spaces.
{% endhint %}

### Basic Expressions

Expressions are combinations of variables, operators, and values that you string together to produce a result. An expression can test values, alter strings, or perform mathematical calculations. Let’s say we have a variable called `COUNT`. In plain English, ...

**to be continued...**

