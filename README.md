# DC/OS Sonic Screwdriver Registry

This repository contains the tools available for the [dcos-sonic-screwdriver](https://github.com/wavesoft/dcos-sonic-screwdriver) tool.

## Entry Description

Each tool has it's own folder in the registry. The name of the folder is the name of the tool, as it's going to be installed in the users computer.

Each tool folder contains at least two files:

* `package.yml` : The general tool information
* `0.0.1.yml` : One or more versions of the tool


```
├── registry/
│   ├── tool-name/
│   │   ├── package.yml  
│   │   └── 0.0.1.yml
```

### `package.yml` - Tool Info

The `package.yml` file contains the general tool information in the following structure:

```yaml
---
desc: "A short (~70 characters) description of this tool"
topics:
  - dcos
  - deployment
help:
  md: yes
  text: |
    Help message for this tool, that could span into multiple lines,
    and could even use *markdown* if you have set `md: yes`
```

* `desc` : A short description for the tool (around 70 characters or less).
* `topics` : An array with one or more keywords related to this tool.
* `help` : A help message, or a link where to find more information about this tool. This will be presented to the user when uses the `ss help <tool>` command.

#### Help Message

You can use three types of help messages:

1. **Embedded Message:** The entire help message is embedded in the registry file. You can optionally set `md: yes` if you want to enable best-effort markdown parsing of the body when presenting it to the user.
    ```yaml
    help:
      md: yes
      text: |
        Multi-line description of the tool
    ```

2. **Remote Message:** Like before, but this time the help message is hosted in a remote location. It will be downloaded and displayed to the user when requested. Like before, you can use `md: yes` to render the contents of the file as markdown.
    ```yaml
    help:
      md: yes
      inline: yes
      url: "http://path/to/remote/README.md"
    ```

3. **Open Browser**: If you want to display a more rich description to the user, you can define a URL to be opened by the user's browser when a help is requested.
    ```yaml
    help:
      url: "http://open/this/link/in/the/browser"
    ```

### `<major>.<minor>.<revision>.yml` - Tool Version

Each tool can have one or more released versions. This can be particular useful if you want to be backwards-compatible and you want to allow the user to install an older version.

Each version record can have one or more **artifacts**. When the tool is about to be installed on the user's machine, the most compatible artifact will be picked and installed. Refer to the [Artifact Types](#artifact-types) for more information about the artifact structure.

A typical version file looks like this:

```yaml
---
artifacts:
  - type: executable
    interpreter:
      shell: bash
    source:
      type: file
      url: https://path/to/artifact.sh
      checksum: 01234567890abcdef01234567890abcdef01234567890abcdef
```

#### Artifact Types

1. **Docker Image Artifact:** The _Sonic Screwdriver_ can automatically wrap docker images as command-line tools.
    ```yaml
      - type: docker
        image: registry/image-name
        tag: "1.2.3"
        dockerArgs: "-p 8080:8080"
        multiCommand:
          ...
    ```

    * `type`: Must be `"docker"`
    * `image`: The docker image to use
    * `tag`: The tag of the docker image to use
    * `require`: _(Optional)_ One or more [Requirements](#requirement-types) that should be satisfied before selecting this artifact.
    * `dockerArgs`: _(Optional)_ Additional arguments to pass on the docker daemon
    * `setupScript`: _(Optional)_ Optional command(s) to run before launching the container.
    * `teardownScript`: _(Optional)_ Optional command(s) to run after launching the container.
    * `command`: _(Optional)_ The command to invoke in the container (defaults to `$*`, that translates to all the arguments the user has given)

2. **Interpreted Artifact:** An interpreted artifact is using a system interpreter for it's execution and therefore is by default platform-agnostic.
    ```yaml
    - type: executable
      interpreter:
        python: python3
        installRequirements: requirements.txt
      source:
        ...
      entrypoint: dcos_ovhcloud_installer.py
      environment:
        VAR: value
    ```

    * `type`: Must be `"executable"`
    * `interpreter`: Describes the interpreter that should be used. Check the [Interpreter Types](#interpreter-types) for more details
    * `source`: Where to download this tool from. Check the [Source Types](#source-types) for more details
    * `entrypoint`: _(Optional)_ If the `source` is an archive-like resource, this field defined where to find the tool
    * `require`: _(Optional)_ One or more [Requirements](#requirement-types) that should be satisfied before selecting this artifact.
    * `installScript`: _(Optional)_ A script to run within the tool directory after downloading it, that can be used to prepare the tool for execution.
    * `environment`: _(Optional)_ one or more environment variables to define before running the script.

2. **Executable Binary:** An executable binary is a machine-dependent artifact and therefore you have to specify the _platform_ and the cpu _architecture_ that it targets.
    ```yaml
    - type: executable
      platform: darwin
      arch: amd64
      source:
        ...
    ```

    * `type`: Must be `"executable"`
    * `platform`: The OS [platform](https://gist.github.com/asukakenji/f15ba7e588ac42795f421b48b8aede63#a-list-of-valid-goos-values) this binary is compatible with (ex. `darwin`, `linux`, `windows`)
    * `arch`: The CPU [architecture](https://gist.github.com/asukakenji/f15ba7e588ac42795f421b48b8aede63#a-list-of-valid-goarch-values) this binary is compatible with (ex. `amd64`)
    * `source`: Where to download this tool from. Check the [Source Types](#source-types) for more details
    * `entrypoint`: _(Optional)_ If the `source` is an archive-like resource, this field defined where to find the tool
    * `require`: _(Optional)_ One or more [Requirements](#requirement-types) that should be satisfied before selecting this artifact.
    * `installScript`: _(Optional)_ A script to run within the tool directory after downloading it, that can be used to prepare the tool for execution.

#### Requirement Types

If an artifact should only be installed if a pre-condition is met, you should describe these preconditions using the `require` key. The following pre-condition checks are available:

1. **Command Must Exists:** Requires that the specified command is available in the user's path:
    ```yaml
    require:
      - cmd: "git"
    ```

2. **Pre-Flight Check:** Requires that the specified command exits with zero:
    ```yaml
    require:
      - exec: "docker --version | grep 18\."
    ```

#### Interpreter Types

The following interpreter types are currently supported by the _Sonic Screwdriver_:

1. **Python:** Runs the script into a python sandbox, using the system's python.
    ```yaml
        interpreter:
          python: python3
          installRequirements: requirements.txt
          installPip: "."
    ```

    * `python`: Must be `"python2"` or `"python3"`
    * `installRequirements`: _(Optional)_ If specified, it should point to a `requirements.txt` file that will be used to install the python dependencies for this tool.
    * `installPip`: _(Optional)_ If specified, it should point to a python package that should be installed with `pip`. You can use `"."` to install your tool and it's dependencies, if your tool is a proper PyPi package.

2. **Shell:** Runs the script using the system's shell interpreter.
    ```yaml
        interpreter:
          shell: bash
    ```

    * `shell`: The name of the shell interpreter to use (ex `"bash"`)

3. **Java:** Runs the tool using the system's Java installation:
    ```yaml
        interpreter:
          java: "1.8"
          javaArgs: "-Xmx1g"
    ```

    * `java`: The minimum version of the JVM required
    * `javaArgs`: Additional arguments to pass to the Java VM

    _Note: If you are using the Java interpreter, the `entrypoint` should point to a .jar archive_


#### Source Types

If you have specified `type: executable` in your artifact, the _Sonic Screwdriver_ will require a `source` field that should point to where the tool should be downloaded from.

There are the following types of sources:

1. **Single File Source:** The tool is hosted somewhere on the web:
    ```yaml
        source:
          type: file
          url: https://path/to/file.sh
          checksum: 01234567890abcdef01234567890abcdef01234567890abcdef
    ```

    * `type`: Must be `"file"`
    * `url`: The URL that points to the file to download
    * `checksum`: The SHA256 checksum to validate the file contents against

2. **Tar Archive Source:** The tool is available inside a `.tar` archive, available somewhere on the web:
    ```yaml
        source:
          type: archive/tar
          url: https://path/to/archive.tar.gz
          checksum: 01234567890abcdef01234567890abcdef01234567890abcdef
        entrypoint: path/in/the/archive
    ```

    * `type`: Must be `"archive/tar"`
    * `url`: The URL that points to the file to download
    * `checksum`: The SHA256 checksum to validate the file contents against
    
    In this source, you should also specify the file in the archive that is the entry-point in the tool:

    * `entrypoint`: The tool path in the archive

3. **Git Repository Source:** The tool is available within a git repository:
    ```yaml
        source:
          type: vcs/git
          url: https://github.com/somename/someproject.git
          branch: master
        entrypoint: path/in/the/repository
    ```

    * `type`: Must be `"vcs/git"`
    * `url`: The URL to the git repository
    * `branch`: _(Optional)_ The branch (or tag) to checkout

    In this source, you should also specify the file in the archive that is the entry-point in the tool:

    * `entrypoint`: The tool path in the archive

## Updating The Registry

The registry can be updated with the sonic-screwdriver's `registry-tool`:

```sh
cd path/to/registry
registry-tool update
```
