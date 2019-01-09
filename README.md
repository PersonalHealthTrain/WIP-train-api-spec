# The Train API V1

**Announcement**: The Documentation of the Train API is now being moved
to this GitBook:

https://lukas-zimmermann.gitbook.io/personal-health-train

The only purpose of this `README.md` is to link to that document. The
content below is no longer being maintained.

# Stale Content

## Requirement

A requirement in the sense of the Train API is a property of the
environment that the train container is executed in. For example, a Train
might require to be executed in a environment (in the sense of an Operating
System) that has an environment variable set. For this the requirement
would look like:

```
{
  "type": "environmentVariable",
  "target": "URL",
  "name": "REQUIRED_VARIABLE_NAME"
}
```
The `type` specifies which further attributes (keys) this requirement
has. In the case here, the `type` `environmentVariable` comes with two
attributes, which are `target` and `name`.

JSON schemata for the requirements should be developed in this repository.
For now, we can say that the allowed values of `type` come from a controlled
vocabulary (CV), which should be reflected in the accompanying JSON schema file.

For now, the list for supported Requirement `type`s is given as follows:

`type`                | Attributes           | Description
----------------------|----------------------|------------------------------------------
`environmentVariable` | `name`, `target`     | States that a environment variable needs to be set in the container. This implies that the OS running inside the container supports environment variables (which all reasonable choices of container images do). For compatibility, an environment variable `name` must match the regex `^[A-Z]+(_[A-Z]+)*$`. There is a CV for denoting the supported `target`s (in my brain, not yet written down).

## Introduction

The Train API defines the set of commands that a train image needs to understand
when a container is created. Each command is characterized by the following
attributes:
* The behavior that the started container will exhibit
* The [OCI runtime config](https://github.com/opencontainers/image-spec/blob/master/config.md)
  options that will alter the behavior
* Additional command-line options specific to the command
* The Output JSON Schema of the command, which is output as a suffix
  of the containers stdout. The output of a running train container in
  this format is called the train response.

Each train image needs to implement the Train API to be valid.

## Train API Specification

### Commands
The Train API V1 supports the following behaviors and commands.

Behavior                        | Command               | Response | Description | OCI Image Config at runtime
--------------------------------|-----------------------|---------------------------------------
List all the resources that the train wants to consume | `list_requirements` | `ListRequirementsResponse` | Lists all the resources that the train wants to consume. This interface can be used to determine the requirements of a train at a very explicit level. | None  
Checks whether all requirements are fullfilled such that the train can run | `check_requirements` | `CheckRequirementsResponse` | The train is supposed to perform a very quick check whether all requirements are met such that the algorithm can run at all.  This might for instance include checking of present environment variables | **config.Env**    
Print a summary of the model    | `print_model_summary` | `PrintModelSummaryResponse` | Print the summary of the model as the train response. This is intended to give the command issuer a brief summary of the state that the model is currently in (like what are the weight s currently) | None
Runs the encapsulated algorithm | `run_algorithm`       | `RunAlgorithmResponse` | Runs the encapsulated algorithm. The resulting exited container contains files that is used to generate the successor train image. | **config.Env**

### Train Responses
Here we list the schema and an example response of each train command.

##### ListRequirementsResponse

###### Examples
```
{
  "requirements": [
    {
      "id": 0,
      "requirement": {
        "type": "environmentVariable",
        "target": "URL",
        "name": "FOO"
      }
    }
  ],
  "relations": [
    {
      "type": "all",
      "requirements": [
        0
      ]
    }
  ]
}
```






<!-- #### print_model_summary
**Schema**
```
{
   "definitions": {},
   "$schema": "http://json-schema.org/draft-07/schema#",
   "$id": "print_model_summary.json",
   "type": "object",
   "title": "Schema for the print_model_summary train response",
   "required": [
     "content"
   ],
   "properties": {
     "content": {
       "$id": "#/properties/content",
       "type": "string",
       "title": "The Content Schema",
       "default": "",
       "examples": [
         "This is some String describing the summary of the model computed so far."
        ],
       "pattern": "^(.*)$"
      }
   }
}
```
**Example**
```
{
  "content": "This is an example summary of a statistical model"
}
```

#### run_algorithm
**Schema**
 -->

## Specific command line interfaces

### Docker
Executing the `docker run` (on version 18.09.0-ce) results in the following
output:
```
"docker run" requires at least 1 argument.
See 'docker run --help'.

Usage:  docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

Run a command in a new container
```
The Train API specifies the set of commands that can be passed to `[COMMAND]`
and the behavior that the started container will then exhibit.

### rkt
Executing `rkt run --help` shows us the following usage:
```
USAGE:
	rkt run [--volume=name,kind=host,...] [--mount volume=VOL,target=PATH] IMAGE [-- image-args...[---]]...
```
Here, the Train API specifies the possible commands that can be passed
to `image-args`.




## API Implementation
### Python
