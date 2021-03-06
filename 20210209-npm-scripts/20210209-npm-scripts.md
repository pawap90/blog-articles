---
title: Mastering NPM Scripts
published: true
description: In this article, I'll share with you my research about how to take full advantage of NPM scripts.
tags: javascript, node, npm
cover_image: https://i.imgur.com/oKamQoX.png
---

You may have come across the `scripts` property in the `package.json` file and even write some scripts yourself. But do you know all you can do with NPM Scripts? 

I've been using NPM Scripts for years, but I wanted to pass a parameter to a script a few weeks ago and realized *I didn't know how to do that*. That's when I decided to learn everything I could about NPM scripts and write this article.

In this article, I'll share my research about how to take full advantage of NPM scripts.

- [Introduction](#introduction)
  - [npm run](#npm-run)
- [Built-in scripts and Aliases](#built-in-scripts-and-aliases)
- [Executing multiple scripts](#executing-multiple-scripts)
- [Understanding errors](#understanding-errors)
- [Run scripts silently or loudly](#run-scripts-silently-or-loudly)
  - [About log levels](#about-log-levels)
- [Referencing scripts from files](#referencing-scripts-from-files)
- [Pre & Post](#pre--post)
- [Access environment variables](#access-environment-variables)
- [Passing arguments](#passing-arguments)
  - [Arguments as environment variables](#arguments-as-environment-variables)
- [Naming conventions](#naming-conventions)
  - [Prefixes](#prefixes)
- [Documentation](#documentation)
- [Conclusion](#conclusion)

---

# Introduction
NPM Scripts are a **set of built-in and custom scripts** defined in the `package.json` file. Their goal is to provide a simple way to **execute repetitive tasks**, like:
- Running a linter tool on your code
- Executing the tests
- Starting your project locally
- Building your project
- Minify or Uglify JS or CSS

You can also use these scripts in your CI/CD pipeline to simplify tasks like build and generate test reports.

To define an NPM script, all you need to do is set its name and write the script in the `script` property in your `package.json` file:

```json
{
    "scripts": {
        "hello-world": "echo \"Hello World\""
    }
}
```

It's important to notice that **NPM makes all your dependencies' binaries available** in the scripts. So you can access them directly as if they were referenced in your PATH. Let's see it in an example:

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

You can also run `npm run`, without specifying a script, to get a **list of all available scripts**:
```
> npm run

Scripts available in sample-project via `npm run-script`:
    hello-world
        echo "Hello World"
```
As you can see, `npm run` prints both the name and the actual script for each script added to the `package.json`.

> ℹ️ `npm run` is an **alias** for `npm run-script`, meaning you could also use `npm run-script hello-world`. In this article, we'll use `npm run <script>` because it's shorter.

# Built-in scripts and Aliases
In the previous example, we created a *custom script* called `hello-world`, but you should know that npm also supports some *built-in scripts* such as `test` and `start`.

Interestingly, unlike our custom scripts, these scripts can be executed using *aliases*, making the complete command **shorter and easier to remember**. For example, all of the following commands will run the `test` script.

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

For these built-in scripts to work, we need to define a script for them in the `package.json`. Otherwise, they will fail. We can write the scripts just as any other script. Here's an example:

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
We may want to **combine** some of our scripts and run them together. To do that, we can use `&&` or `&`.

- To run multiple scripts **sequentially**, we use `&&`. For example: `npm run lint && npm test`
- To run multiple scripts **in parallel**, we use `&`. Example: `npm run lint & npm test`
  - This only works in Unix environments. In Windows, it'll run sequentially.

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
When a script finishes with a **non-zero exit code**, it means an **error** occurred while running the script, and the execution is terminated.

That means we can purposefully end the execution of a script with an error by exiting with a non-zero exit code, like so:

```json
{
    "scripts": {
        "error": "echo \"This script will fail\" && exit 1"
    }
}
```

When a script throws an error, we get a few other details, such as the error number `errno` and the `code`. Both can be useful for googling the error.

And if we need more information, we can always access the complete log file. The path to this file is provided at the end of the error message. **On failure, all logs are included in this file.**

# Run scripts silently or loudly
Use `npm run <script> --silent` to **reduce logs** and to **prevent the script from throwing an error**. 

The `--silent` flag (short for `--loglevel silent`) can be helpful when you want to run a script that you know may fail, but you don't want it to throw an error. Maybe in a CI pipeline, you want your whole pipeline to keep running even when the `test` command fails. 

It can also be used as `-s`: `npm run <script> -s`

> ℹ️ If we don't want to get an error when the script *doesn't exists*, we can use `--if-present` instead: `npm run <script> --if-present`.

## About log levels
We saw how we can reduce logs using `--silent`, but what about getting even **more detailed logs**? Or something in between? 

There are different *log levels*: "silent", "error", "warn", "notice", "http", "timing", "info", "verbose", "silly". The default is "notice". The log level determines **which logs will be displayed** in the output. Any logs of a higher level than the currently defined will be shown. 

We can explicitly define which loglevel we want to use when running a command, using `--loglevel <level>`. As we saw before, the `--silent` flag is the same as using `--loglevel silent`. 

Now, if we want to get more detailed logs, we'll need to use a higher level than the default ("notice"). For example: `--loglevel info`. 

There are also short versions we can use to simplify the command:
- `-s`, `--silent`, `--loglevel silent`
- `-q`, `--quiet`, `--loglevel warn`
- `-d`, `--loglevel info`
- `-dd`, `--verbose`, `--loglevel verbose`
- `-ddd`, `--loglevel silly`

So to get the highest level of detail we could use `npm run <script> -ddd` or `npm run <script> --loglevel silly`.

# Referencing scripts from files
You can execute scripts from files. This can be useful for especially *complex scripts* that would be hard to read in the `package.json` file. However, it doesn't add much value if your script is short and straightforward.

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
We use `node <script-path.js>` to execute JS files and `bash <script-path.sh>` to execute bash files.

Notice that you can't just call `scripts/helloworld.cmd` for CMD and BAT files. You'll need to navigate to the folder using `cd` first. Otherwise, you'll get an error from NPM.

Another advantage of executing scripts from files is that, if the script is complex, it'll be easier to maintain in a separate file than in a single line inside the `package.json` file. 

# Pre & Post
We can create "pre" and "post" scripts for *any of our scripts*, and NPM will automatically **run them in order**. The only requirement is that the script's name, following the "pre" or "post" prefix, matches the main script. For example:

```json
{
    "scripts": {
        "prehello": "echo \"--Preparing greeting\"",
        "hello": "echo \"Hello World\"",
        "posthello": "echo \"--Greeting delivered\""
    }
}
```
If we execute `npm run hello`, NPM will execute the scripts in this order: `prehello`, `hello`, `posthello`. Which will result in the following output:

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

> ℹ️ If we run `prehello` or `posthello` individually, NPM ***will not*** automatically execute any other scripts. It only works if you run the "main" script, in this case, `hello`.

# Access environment variables

While executing an NPM Script, NPM makes available a set of *environment variables* we can use. These environment variables are generated by taking data from NPM Configuration, the package.json, and other sources.

**Configuration** parameters are put in the environment using the `npm_config_` prefix. Here are a few examples:

```json
{
    "scripts": {
        "config:loglevel": "echo \"Loglevel: $npm_config_loglevel\"",
        "config:editor": "echo \"Editor: $npm_config_editor\"",
        "config:useragent": "echo \"User Agent: $npm_config_user_agent\""
    }
}
```

Let's see what we get after executing the above commands:

```sh
> npm run config:loglevel
# Output: "Loglevel: notice"

> npm run config:editor
# Output: "Editor: notepad.exe"

> npm run config:useragent
# Output: "User Agent: npm/6.13.4 node/v12.14.1 win32 x64"
```


> ℹ️ You can also run `npm config ls -l` to get a **list of all the configuration parameters** available.

Similarly, **`package.json` fields**, such as `version` and `main`, are included with the `npm_package_` prefix. Let's see a few examples:

```json
{
    "scripts": {
        "package:main": "echo \"Main: $npm_package_main\"",
        "package:name": "echo \"Name: $npm_package_name\"",
        "package:version": "echo \"Version: $npm_package_version\""
    }
}
```

The results from these commands will be something like this:

```sh
> npm run package:main
# Output: "Main: app.js"

> npm run package:name
# Output: "Name: npm-scripts-demo"

> npm run package:version
# Output: "Version: 1.0.0"
```

Finally, you can add **your own environment variables** using the *`config` field* in your `package.json` file. The values setup there will be added as environment variables using the `npm_package_config` prefix. 

```json
{
    "config": {
        "my-var": "Some value",
        "port": 1234
    },
    "script": {
        "packageconfig:port": "echo \"Port: $npm_package_config_port\"",
        "packageconfig:myvar": "echo \"My var: $npm_package_config_my_var\""
    }
}
```

If we execute both commands we'll get:

```sh
> npm run packageconfig:port
# Output: "Port: 1234"

> npm run packageconfig:myvar
# Output: "My var: Some value"
```

> ℹ️ In Windows' `cmd` instead of `$npm_package_config_port` you should use `%npm_package_config_port%` to access the environment variables.

# Passing arguments
In some cases, you may want to pass some **arguments** to your script. You can achieve that using `--` that the end of the command, like so: `npm run <script> -- --argument="value"`.

Let's see a few examples:
```json
{
    "scripts": {
        "lint": "eslint .",
        "test": "jest ./test",
    }
}
```

If I wanted to run only the tests that changed, I could do this: 

``` sh
> npm run test -- --onlyChanged
``` 

And if I wanted to run the linter and save the output in a file, I could execute the following command: 

```sh
> npm run lint -- --output-file lint-result.txt
```

## Arguments as environment variables
Another way of **passing arguments** is **through environment variables**. Any key-value pairs we add to our script will be translated into an environment variable with the `npm_config` prefix. Meaning we can create a script like this:

```json
{
    "scripts": {
        "hello": "echo \"Hello $npm_config_firstname!\""
    }
}
```

And then use it like so: 
```sh
> npm run hello --firstname=Paula
# Output: "Hello Paula"
```

# Naming conventions

There are no specific guidelines about how to name your scripts, but there are a few things we can keep in mind to make our scripts easier to pick up by other developers.

Here's my take on the subject, based on my research:
- Keep it **short**: If you take a look at Svelte's NPM Scripts, you'll notice that most script names are *one word only*. If we can manage to keep our script names short, it'll be easier to remember them when we need them.
- Be **consistent**: You may need to use more than one word to name your script. In that case, choose a *naming style and stick to it*. It can be camelCase, kebab-case, or anything you prefer. But avoid mixing them. 

## Prefixes
One convention that you may have seen is using a **prefix and a colon to group scripts**, for example, "build:prod". This is simply a naming convention. It doesn't affect your scripts' behavior but can be helpful to create groups of scripts that are *easier to identify by their prefixes*.

Example: 
```json
{
    "scripts": {
        "lint:check": "eslint .",
        "lint:fix": "eslint . --fix",
        "build:dev": "...",
        "build:prod": "..."
    }
}

```

# Documentation
Consider adding documentation for your scripts so other people can easily understand *how and when to use them*. I like to add a few lines explaining each script on my Readme file.  

The documentation for each available script should include:
- Script name
- Description
- Accepted arguments (optional)
- Links to other documentation (optional): For example, if your script runs `tsc --build`, you may want to include a link to Typescript docs.

# Conclusion
This is all I managed to dig up about NPM Scripts. I hope you find it useful! I certainly learned a lot just by doing this research. It took me way more time than I thought it would, but it was totally worth it.

Let me know if there's anything missing that you'll like to add to make this guide even more complete! 💬