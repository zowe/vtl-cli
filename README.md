# Velocity Template Language (VTL) Command-Line Interface

[![codecov](https://codecov.io/github/zowe/vtl-cli/branch/master/graph/badge.svg)](https://codecov.io/github/zowe/vtl-cli)

This is simple Java command-line application that uses Apache Velocity to 'merge' VTL templates from shell scripts. 

Features:
- Apache Velocity 1.7 template engine
- Output to the console or to a file
- Context variables can be provided in:
  - Command line parameters
  - Environment variables
  - YAML file
- Configurable encoding for input and output
- One small fully executable JAR file (no `java -jar...` on Linux and z/OS)
- Works everywhere where is Java 

## Build

```
./gradlew build
```

## Installation

1. Download https://github.com/zowe/vtl-cli/releases/download/v1.0.0/vtl.tar.gz:

        curl -LO https://github.com/zowe/vtl-cli/releases/download/v1.0.0/vtl.tar.gz

    or locate new release of vtl.tar.gz in https://github.com/zowe/vtl-cli/releases
2. Extract it:

        unzip vtl.tar.gz

3. There are several files available:
    
    - `vtl-cli.jar` for execution by the `java -jar vtl-cli.jar` command
    - `vtl` for execution by the `vtl` command on Linux systems with `java` in `PATH` or `JAVA_HOME` set
    - `zos/vtl` for execution by the `vtl` command on z/OS systems

## Usage

```
java -jar build/vtl-cli.jar templates/hello.vtl -c name=world
```

or 

```
vtl templates/hello.vtl -c name=world
```

If the `hello.vtl` file contains:

    Hello, ${name}!

Then the standard output of `vtl-cli` is:

    Hello, world!

## Syntax

```
vtl [-e] [-ie=<inputEncoding>] [-o=<outputFile>] [-oe=<outputEncoding>] [-y=<yamlContextFile>] [-ye=<yamlEncoding>]
    [-c=variable=value]... FILE

Parameters:
      FILE                 File with a Velocity template to process

Options:
      -ie, --input-encoding=<inputEncoding>
                           UTF8, ISO8859-1, Cp1047, ... - see https://goo.gl/yn2pJZ
  -c, --context=variable=value
                           Context variable for Velocity (can be repeated)
  -y, --yaml-context=<yamlContextFile>
                           YAML file with context variables
      -ye, --yaml-encoding=<yamlEncoding>
                           UTF8, ISO8859-1, Cp1047, ...
  -e, --env-context        Set the context variables from environment
  -o, --out=<outputFile>   Output file (default: print to console)
      -oe, --output-encoding=<outputEncoding>
                           UTF8, ISO8859-1, Cp1047, ...
```

## Loading context from YAML file

You can write context variables into a YAML file - for example:

```yaml
name: world
```

and the use it:

```
vtl --yaml-context templates/hello.yml templates/hello.vtl
```

If you have nested YAML properties like:

```yaml
nested:
    name: world
```

you can references it as `${nested.name}`.

## VTL

The template language is described in [Velocity User Guide](http://velocity.apache.org/engine/1.7/user-guide.html).

## Releasing

The new releases are done by Jenkins after new tag is created:

    git tag v1.0.0
    git push --tags

