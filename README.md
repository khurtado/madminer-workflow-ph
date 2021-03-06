# Madminer workflow PH


## About
This repository defines a Physics workflow using the [Madminer package][madminer-repo]
as the base for generating events. To learn more about Madminer and its ML tools check
the [Madminer documentation][madminer-docs].

The workflow generates a file with _Pythia generated_ - _Delphes showered_ events as output.
This file can be later on used in the [Madminer ML workflow][madminer-workflow-ml],
which originally was a linked workflow after executing this one.


## Workflow definition
The workflow specification is composed of 3 hierarchical layers. From top to bottom:

1. **Workflow spec:** description of how steps are coordinated.
2. **Shell scripts:** entry points for each of the steps.
3. **Python scripts:** set of actions to interact with Madminer.

The division into multiple layers is very useful for debugging. It provides developers an easy way 
to test individual steps before testing the full workflow coordination.

Considering the workflow steps:

                        +--------------+
                        |  Configurate |
                        +--------------+
                                |
                                |
                                v
                        +--------------+
                        |   Generate   |
                        +--------------+
                                |
            +-------------------+-------------------+
            |                   |                   |
            v                   v                   v
    +--------------+    +--------------+    +--------------+
    |   Pythia_0   |    |   Pythia_1   |    |      ...     |
    +--------------+    +--------------+    +--------------+
            |                   |                   |
            |                   |                   |
            v                   v                   v
    +--------------+    +--------------+    +--------------+
    |   Delphes_0  |    |   Delphes_1  |    |      ...     |
    +--------------+    +--------------+    +--------------+
            |                   |                   |
            +-------------------+-------------------+
                                v
                        +--------------+
                        |    Combine   |
                        +--------------+


## Installation
For executing locally executing the steps of this workflow several non-trivial dependencies
need to be installed.

Follow the [installation guide][install-guide] to have your local environment ready.


## Execution
When executing the workflow (either fully or some of its parts) it is important to consider that
each individual step received inputs and generates outputs. Outputs are usually files, which need
to be temporarily stored to serve as input for later steps.

The _shell script layer_ provides an easy way to redirect outputs to what is called a `WORKDIR`.
The `WORKDIR` is just a folder where steps output files are stored to be used for other ones.
In addition, this layer provides a way of specifying the path where the project code and scripts 
are stored. This becomes useful to allow both _local_ and _within-docker_ executions.

When executing the workflow there are 2 alternatives:

### A) Individual steps
Individual steps can be launched using their shell script. Be aware their execution may depend on 
previous step outputs, so a sequential order must be followed.

Example:
```shell script
scripts/1_configurate.sh \
    --project_path . \
    --input_file workflow/input.yml \
    --output_dir .workdir
```

### B) Coordinated
The full workflow can be launched using [Yadage][yadage-repo]. Yadage is a YAML specification language
over a set of utilities that are used to coordinate workflows. Please consider that it can be hard
to define Yadage workflows as the [Yadage documentation][yadage-docs] is incomplete.
For learning about Yadage hidden features contact [Lukas Heinrich][lukas-profile], Yadage creator.

Yadage depends on having a Docker image used as environment already published. For publishing the Docker
image for this workflow jump to the [Docker section](#docker).

Once the Docker image is published:
```shell script
pip3 install yadage
make yadage-run
```


## Docker
To build a new Docker image for local testing:
```shell script
make build
```

To publish a new Docker image, bump up the `VERSION` number and execute:

```shell script
export DOCKERUSER=<your_dockerhub_username>
export DOCKERPASS=<your_dockerhub_password>
make push
```


[install-guide]: INSTALL.md
[madminer-docs]: https://madminer.readthedocs.io/en/latest/index.html
[madminer-repo]: https://github.com/diana-hep/madminer
[madminer-workflow-ml]: https://github.com/scailfin/madminer-workflow-ml
[yadage-repo]: https://github.com/yadage/yadage
[yadage-docs]: https://yadage.readthedocs.io/en/latest/
[lukas-profile]: https://github.com/lukasheinrich
