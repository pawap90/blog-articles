---
title: Mastering NPM Scripts
published: false
description: In this article I'll share with you my research about how to take full advantage of NPM scripts.
tags: javascript, node, npm
cover_image: https://i.imgur.com/oKamQoX.png
---

You may have come across the `scripts` property in the `package.json` file, and even write some scripts yourself. But do you know all you can do with NPM Scripts? 

I've been using NPM Scripts for years but a few weeks ago I wanted to pass a parameter to a script and realized I didn't know how to do that. That's when I decided to learn everything I could about NPM scripts and write this article.

In this article I'll share with you my research about how to take full advantage of NPM scripts.

- [Introduction](#introduction)
  - [npm run](#npm-run)
- [Built-in scripts and Aliases](#built-in-scripts-and-aliases)
- [Executing multiple scripts](#executing-multiple-scripts)
- [Understanding errors](#understanding-errors)
- [Run scripts silently 🤫](#run-scripts-silently-)
  - [About Loglevels](#about-loglevels)
- [Referencing scripts from files](#referencing-scripts-from-files)
- [Pre & Post](#pre--post)
- [Access environment variables](#access-environment-variables)
- [Passing arguments](#passing-arguments)
- [Naming conventions](#naming-conventions)
  - [Prefixes](#prefixes)
- [Documentation](#documentation)


# Introduction
NPM Scripts are a set of built-in and custom scripts defined in the `package.json` file. Their goal is to provide a simple way to execute repetitive tasks, like running a linter or executing the tests on your project. like
- Running a linter tool on your code
- Executing the tests
- Starting your project locally
- Building your project
- Minify or Uglify JS or CSS

You can also use these scripts in your CI/CD pipeline to simplify tasks like build and generate test reports.

To define a NPM script all you need to do is set it's name and write the script in the `script` property in your `package.json` file:

```json
{
    "scripts": {
        "hello-world": "echo \"Hello World\""
    }
}
```

It's important to notice that NPM makes all your dependencies' binaries available in the scripts. So you can access them directly as if they were referenced in your PATH. Let's see it in an example:

Instead of doing this:
```json
{
    "scripts": {
        "lint": "./node_modules/.bin/eslint .",
    }
}
```

You can do this:
```json
{
    "scripts": {
        "lint": "eslint ."
    }
}
```

## npm run
Now all you need to do is run `npm run hello-world` on the terminal from your project's root folder.

```sh
> npm run hello-world

"Hello World"
```

You can also run `npm run` to get a list of all available scripts:
```
> npm run

Scripts available in sample-project via `npm run-script`:
    hello-world
        echo "Hello World"
```
As you can see, `npm run` prints both the name and the actual script for each script added to the `package.json`.

> ℹ️ `npm run` is an alias for `npm run-script`, meaning you could also use `npm run-script hello-world`. In this article, we'll use `npm run <script>` because it's shorter.

# Built-in scripts and Aliases
In the previous example we created a *custom script* called `hello-world`, but that npm also supports a number of *built-in scripts* such as `test` and `start`.

What's interesting about these scripts is that, unlink our custom scripts, they can be executed using *aliases* which make the complete command shorter and easier to remember. For example, all of the following commands will run the `test` script.

```sh
npm run-script test
npm run test
npm test
npm t
```

Similarly to the `test` command, all of the following will run the `start` command:

```sh
npm run-script start
npm run start
npm start
```

For this built-in scripts to work, we need to define a script for them in the `package.json`, otherwise they will fail. We can write the scripts just as any other script. Here's an example:

```json
{
    "scripts": {
        "start": "node app.js",
        "test": "jest ./test",
        "hello-world": "echo \"Hello World\""
    }
}
```

# Executing multiple scripts
We may want to combine some of our scripts and run them together. To do that we can use `&&` or `&`.

- To run multiple scripts sequentially, we use `&&`. For example: `npm run lint && npm test`
- To run multiple scripts in parallel, we use `&`. Example: `npm run lint & npm test`
  - This only works in Unix environments. In Windows it'll run sequentially.

So, for example, we could create a script that combines two other scripts, like so:
```json
{
    "scripts": {
        "lint": "eslint .",
        "test": "jest ./test",
        "ci": "npm run lint && npm test"
    }
}
```

# Understanding errors
When a script finishes with **non-zero exit code**, it means an error ocurred while running the script and the execution is terminated.

That means we can purposfully end the execution of a script with an error by exiting with a non-zero exit code, like so:

```json
{
    "scripts": {
        "error": "echo \"This script will fail\" && exit 1"
    }
}
```

When a script throws an error we get a few other details such as the error number `errno` and the `code`. Both can be useful for googling the error.

And if we need more information, we can always access the complete log file. The path to this file is provided at the end of the error message. On failure, all logs are included in this file.

# Run scripts silently 🤫
Use the `npm run <script> --silent` to **reduce logs** and to **prevent the script from throwing an error**. 

The `--silent` flag (short for `--loglevel silent`) can be useful when you want to run a script that you know may fail, but you don't want it to throw an error. Maybe in a CI pipeline you want your whole pipeline to keep running even when the `test` command fails. 

It can also be used as `-s`: `npm run <script> -s`

> If we dont' want to get an error when the script doesn't exists, we can use `--if-present` instead: `npm run <script> --if-present`.

## About Loglevels
We saw how we can reduce logs using `--silent` but what about getting even more detailed logs? Or something in between? 

There are different log levels: "silent", "error", "warn", "notice", "http", "timing", "info", "verbose", "silly". The default is "notice". The log level determines which logs will be displayed in the output. Any logs of higher level than the currently defined will be shown. 

We can explicitly define which loglevel we want to use when running a command, using `--loglevel <level>`. As we saw before, the `--silent` flag is the same as using `--loglevel silent`. 

Now, if we want to get more detailed logs we'll need to use a higher level than the default ("notice"). For example: `--loglevel info`. 

There are also short versions we can use to simplify the command:
- `-s`, `--silent`, `--loglevel silent`
- `-q`, `--quiet`, `--loglevel warn`
- `-d`, `--loglevel info`
- `-dd`, `--verbose`, `--loglevel verbose`
- `-ddd`, `--loglevel silly`

So to get the higher level of detail we could use `npm run <script> -ddd` or `npm run <script> --loglevel silly`.

# Referencing scripts from files
You can execute scripts from files. This can be useful for specially complex scripts. However, it doesn't add much value if your script is short and simple.

Consider this example:
```json
{
    "scripts": {
        "hello:js": "node scripts/helloworld.js",
        "hello:bash": "bash scripts/helloworld.sh",
        "hello:cmd": "cd scripts && helloworld.cmd"
    }
}
```
We use `node <script-path>` to execute JS files and `bash <script-path>` to execute bash files.

Notice that for CMD and BAT files you can't just call `scripts/helloworld.cmd`. You'll need to navigate to the folder using `cd` first, otherwise you'll get an error from NPM.

Another advantage from executing scripts from files is that, if the script is complex, it'll be easier to maintain in a separate file than in a single line inside the `package.json` file. 

# Pre & Post
We can create "pre" and "post" scripts for any of our scripts and NPM will automatically run them in order. The only requirement is that the name of the script, following the "pre" or "post" prefix, matches the main script. For example:

```json
{
    "scripts": {
        "prehello": "echo \"--Preparing greeting\"",
        "hello": "echo \"Hello World\"",
        "posthello": "echo \"--Greeting delivered\""
    }
}
```
If we execute `npm run hello` NPM will execute the scripts in this order: `prehello`, `hello`, `posthello`. Which will result in the following output:

```
> script-test@1.0.0 prehello
> echo "--Preparing greeting"

"--Preparing greeting"

> script-test@1.0.0 hello
> echo "Hello World"

"Hello World"

> script-test@1.0.0 posthello
> echo "--Greeting delivered"

"--Greeting delivered"
```

> If we run `prehello` or `posthello` individually, NPM will not execute any other scripts automatically. It only works if you run the "main" script, in this case `hello`.

# Access environment variables

# Passing arguments
In some cases you may want to pass some arguments to your script. You can achieve that using `--` that the end of the command, like so: `npm run <script> -- --argument="value"`.

Let's see a few examples:
```json
{
    "scripts": {
        "lint": "eslint .",
        "test": "jest ./test",
    }
}
```

If I wanted to run only the tests that changed I could do this: `npm run test -- --onlyChanged`.
And if I wanted to run the linter and save the ouput in a file I could execute the following command: `npm run lint -- --output-file lint-result.txt`.

# Naming conventions

There are no specific guidelines about how to name your scripts, but there are a few things we can keep in mind in order to make our scripts easier to pick up by other developers.

Here's my take on the subject, based on my research:
- Keep it **short**: If you take a look at Svelte's NPM Scripts you'll notice that most script names are one word only. If we can manage to keep our script names short it'll be easier to remember them when we need them.
- Be **consistent**: You may need to use more than one word to name your script. In that case choose a style and stick to it. It can be camelCase or you can use hyphens to separate words. But avoid mixing them.

## Prefixes
One convention that you may have seen is using a prefix and a colon to group scripts. This is simply a naming convention, it doesn't affect the behaviour of your scripts, but can be useful to create groups of scripts that are easier to identify by their prefixes.

Example: 
```json
{
    "scripts": {
        "lint:check": "eslint .",
        "lint:fix": "eslint . --fix"
    }
}

```

# Documentation
Consider adding documentation for your scripts so other people can easily understand how and when to use them. I like to add a few lines explaining each script on my Readme file.  

The documentation for each available script should include:
- Script name
- Description
- Accepted arguments (optional)
- Links to other documentation (optional): For example, if your script runs `tsc --build` you may want to include a link to Typescript docs.