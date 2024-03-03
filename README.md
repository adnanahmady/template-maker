# Config Maker

This framework helps you to create different files based on your specified templates.

## Development Process

The development process mentioned in this section is just a recommended way to do things,
but it's not mandatory, and you are free to develop as you will.

**Important** the files need to have executable permission, to do such a thing
you can run `chmod` command on your added file like this `chmod +x ./my-file`.

**Important** It's strongly recommended to use `command` environment variables
for unifying the command names throughout the project, you can do this by adding
your commands in `./utils/commands` file like bellow.

```shell
#! /bin/sh

PROXY_COMMAND=hello
export PROXY_COMMAND
```

### Step 1

A template in `./templates/` directory. like bellow.

`./templates/hello.template`
```
Hello dear "${GREETED_USER}", You are in ${PWD} path.
Hope you like this.

Best regards
${MESSAGE_WRITER}
```

In this template we have three environment variables listed as
`GREETED_USER`, `PWD`, and `MESSAGE_WRITER`, that these variables
will be replaced with user input if you want.

** Pay attention ** The `PWD` environment variable may already be set
in your operating system, so if you dont over write it, it will be 
replaced by your operating systems environment variable.

### Step 2

Add an action for this template in `./actions/` directory, add a check action
for the command like bellow.

`./actions/hello-action`
```shell
#! /bin/sh

. "${PROJECT_PATH:-..}"/utils/functions

if [ "$1" = "${HELLO_COMMAND:-hello}" ]; then
  "${PROJECT_PATH:-..}"/controllers/hello-controller "$2"
  stopGoingToOtherHandlers
fi
```

### Step 3

You should add the controller at this step, add `hello-controller`
to `./controllers/` directory like bellow.

`./controllers/hello-controller`
```shell
#! /bin/sh

. "${PROJECT_PATH:-$(dirname "$0")/..}"/utils/printer

TEMPLATE_PATH="${PROJECT_PATH:-$(dirname "$0")/..}"/templates/hello.template
OUTPUT_PATH=$1
INPUTS="Greeted user;Message writer"

if [ -z "$OUTPUT_PATH" ]; then
  OUTPUT_PATH='Output path'
  INPUTS="${OUTPUT_PATH};${INPUTS}"
fi

"${PROJECT_PATH:-$(dirname "$0")/..}"/lib/template-resolver "${TEMPLATE_PATH}" "${OUTPUT_PATH}" "${INPUTS}"

printInLightGreen "A configuration file created in \"${OUTPUT_PATH}\" path."
```

### Step 4

You can run the command that you just build to see the results.

Assuming you enter the command like `./maker hello ./output.txt` 
you will face an interface like this.

```shell
$ ./maker hello ./output.txt
```
```
Greeted user: David
Message writer: John
A configuration file created in "./output.txt" path.
```

The resulted file will be like bellow.

```
Hello dear "David", You are in /home/user/template-maker path.
Hope you like this.

Best regards
John
```

### Adding Help information

You can add a help information for the command like bellow.

`./docs/hello-doc`
```shell
#! /bin/sh

. "${PROJECT_PATH:-$(dirname "$0")/..}"/utils/printer

printInWhite "${HELLO_COMMAND:-hello}"
space
printInGreen "Creates a message for you"
newLine
tab
printInBlue "outputPath"
space
printInLightGreen "This argument has no name and you should only path the value."
```

And the hello command will have a help documentation to show.

you can see the help information by only running `./maker` command.

```shell
$ ./maker

Help information

hello Creates a message for you
	outputPath This argument has no name and you should only path the value.
make-proxy Creates a nginx proxy configuration for you at the specified output path.
	outputPath This argument has no name and you should only path the value.
```

## Structure

```
- actions/
    - reverse-proxy-config-file-action-checker
- controllers/
    - reverse-proxy-config-file-maker
- docs/
    - reverse-proxy-config-file-action-document
- lib/
    - make-logics
- templates/
    - reverse-proxy-template
- utils/
    - helpers
- maker
```

### Actions

You can add your action check in this section.
For example for checking if `make-proxy` command is
requested you do an action like bellow.

`./actions/make-proxy-action`
```shell
#! /bin/sh

. "${PROJECT_PATH:-..}"/utils/functions

if [ "$1" = "${PROXY_COMMAND}" ]; then
  "${PROJECT_PATH:-..}"/controllers/proxy-controller "$2"
  stopGoingToOtherHandlers
fi
```

### Controllers

You can do things for request in this section. like bellow.

`./controllers/proxy-controller`
```shell
#! /bin/sh

. "${PROJECT_PATH:-$(dirname "$0")/..}"/utils/printer

TEMPLATE_PATH="${PROJECT_PATH:-$(dirname "$0")/..}"/templates/proxy.template
OUTPUT_PATH=$1
INPUTS="server names;public crt path;private crt path;upstream;log"

"${PROJECT_PATH:-$(dirname "$0")/..}"/lib/template-resolver "${TEMPLATE_PATH}" "${OUTPUT_PATH}" "${INPUTS}"

printInLightGreen "A configuration file created in \"${OUTPUT_PATH}\" path."
```

### Documentations

You can add help documentation in this section. like bellow.
The document will be shown for helping user.

`./docs/proxy-documentation`
```shell
#! /bin/sh

. "${PROJECT_PATH:-$(dirname "$0")/..}"/utils/printer

printInWhite "${PROXY_COMMAND:-proxy-maker}"
space
printInGreen "Creates a nginx proxy configuration for you at the specified output path."
newLine
tab
printInBlue "outputPath"
space
printInLightGreen "This argument has no name and you should only path the value."
```

### Templates

You can put your predefined templates in this section. like bellow.

`./templates/proxy.template`
```shell
server {
    listen 443 ssl;
    server_name ${SERVER_NAMES};

    ssl_certificate ${PUBLIC_CRT_PATH};
    ssl_certificate_key ${PRIVATE_CRT_PATH};

    location / {
        proxy_pass https://${UPSTREAM};
    }
}
```

### Utils

This section is for helping you to develop your command.

### Libraries

You can put your libraries in `lib` section.

### Bootstrap

In this section bootstrapper code is placed.
