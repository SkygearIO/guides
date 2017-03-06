# Skygear Guides

This is the repository and issue trackers for Skygear Guides. The markdown files
are used to generate [docs.skygear.io](https://docs.skygear.io) with
[skygeario/skygear-doc](https://github.com/SkygearIO/skygear-doc).

## Creating alert boxes

You can use the following syntax in a markdown file to create an "Advanced" box:

```
::: advanced

Some advanced usage

:::
```

Besides `advanced`, you can also use `todo`, `note`, `tips` and `caution`.

`todo` is a special type of box to warn users the section is a WIP. It should
always include a bullet points list of todo items.

## Creating code switcher (e.g. Objective-C vs Swift)

Any two consecuting fenced code blocks of different languages will be combined
as a code switcher, e.g.

    ``` objectivec
    some Objective-C code
    ```

    ``` swift
    some swift code
    ```

Note: Currently it only supports two languages; the behaviour will be unexpected
if there are more consecutive different language fenced code blocks.
You can insert paragraphs and text between the codes to avoid this.

## Style Guide

- Use You to refer to the reader
- Include external link (e.g. webpack) as necessary but not to be repetitive
- Links used in the markdown can be all placed at the bottom of the file
- Avoid starting a sentence with a verb
- Notice the difference between a method and a function, e.g. `skygear.loginWithProvider` is a method
- Start heading at `h2` (`h1` is for page title from route), and all `h2` has to be as sub-menu item
- Use &lt;brackets> for parts that require user to fill in
- Sample user input should be highlighted. (Refer to getting started for JS new project yeoman terminal dump as example)
- Code inline comment should only be used for simple remarks. Long descriptions should be written as separate paragraphs
- While there are spelling check for paragraphs, there are none for code. Therefore please double check the spellings in code blocks
- Use terms consistently especially in titles. Don't shorten words - readers will be confused
- Avoid saying too much about the server in each of the SDK doc, because:
  1. it will be repetitive
  2. technically those descriptions are not directly related to the SDK itself

  Instead you can link to the relevant section in the server doc.
  However talking about server behavior in each of the SDK is necessary although it is repetitive
  e.g. in Users & Auth, you need to talk about Skygear not allowing duplicated usernames

## Style Guide for JS Docs
- Two spaces per indent
- Use `console.error(error)` instead of `console.log(err)`
- all object properties should be quoted
- use ES6 syntax, ES5 syntax will be added later as code tab switching is enabled

