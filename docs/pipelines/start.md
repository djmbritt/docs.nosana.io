# Getting Started

Now that you have a basic understanding of what pipelines are, let's get started with a simple example.
As mentioned in the previous page, we will create a pipeline that is used to create this documentation site.
This site is created with [vuepress](https://vuepress.vuejs.org/), a static site generator for vue.js.
It's great for creating documentation sites, and it's pretty straightforward to get started with.

You can find the source code for this site on [gitlab](https://gitlab.com/nosana-ci/nosana-docs).

To start off this journey we'll have to take a look at the `.nosana-ci.yml` file that is used to configure this pipeline.
You should be able to find it in the root of the repository.

Yaml is a human readable data serialization format that is commonly used for configuration files.
It's easy to read and write, and it's supported by most programming languages.
It's the standard for configuring ci/cd pipelines on gitlab, github and other popular code hosting platforms.
You can find more information about yaml [here](https://yaml.org/).

Let's take a look at the `.nosana-ci.yml` file for this site:
To begin with there are 3 top level definitions that are used to configure the pipeline: `nosana`, `global`, and `jobs`.
The `nosana` section is used to configure the pipeline itself, and the `global` section is used to configure the environment that the pipeline will run in.
The `jobs` section is used to configure the individual jobs that will be executed in the pipeline.

::: tabs
@tab nosana

## nosana

We'll explore the `nosana` section first.

```yaml
# .nosana-ci.yml
nosana:
    # the name of the nosana pipeline
    description: build and deploy the nosana documentation site
    # backend storage service
    storage: ipfs # (default)
...

```

- `description`: the name of the pipeline. this is used to identify the pipeline in the nosana network.
<!-- IDEA add webhook url in nosana.yml file to post data somewhere  -->
<!-- # webhook used to send data to when the pipeline is finished -->
<!-- webhook: https://nosana.com/api/v1/webhooks/... -->
<!-- - `webhook`: the url of the webhook that will be used to send data to when the pipeline is finished. -->
- `storage`: the backend storage service that will be used to store the data that is generated by the pipeline.

@tab global

## global

The `global` section is used to configure the environment that the pipeline will run in.
this includes the image that will be used to run the pipeline, and the environment variables that will be available to the pipeline.
it is possible to define the image in the `global` section, or in the `jobs` section.
this is useful if you want to use a different image for each job. if the image is not defined in the jobs section, the image defined in the `global` section will be used.

```yaml
...

# global settings
global:
    # image used to run the pipeline
    image: registry.hub.docker.com/library/node:16

    # event that will trigger the pipeline.
    trigger:
        branch:
            - main
            - develop

    # environment variables that will be available to the pipeline
    environment:
        APP_ENV: production
        APP_DEBUG: false

    # allow the pipeline to abort if one of the commands returns a failure state.
    allow_failure: true # false
...
```

- `market`: the solana address of the nosana market that will be used to pay for the pipeline.
- `image`: the image that will be used to run the pipeline. this can be any docker image that is available on docker hub. the image will be pulled from docker hub when the pipeline is executed. If the image is not defined in the `jobs` section, the image defined in the `global` section will be used. Note the format of the resource identifier used when pulling down an image: `registry.hub.docker.com/user/image:tag` or `docker.io/user/image:tag`. More information about docker images can be found [here](https://docs.docker.com/registry/introduction/).
- `trigger`: the event that will trigger the pipeline. this can be a branch, tag, or a schedule.
  - `branch`: the pipeline will be triggered when a commit is created on the specified branch.
  <!-- TODO - `tag`: the pipeline will be triggered when a tag is created. -->
- `environment`: environment variables that will be available to the pipeline.
- `allow_failure`: allow the pipeline to abort if one of the commands returns a failure state.

Now that we've managed to get a basic understanding of the `.nosana-ci.yml` file, let's take a look at the individual jobs that are defined in the `jobs` section.

@tab jobs

## jobs

The `jobs` section is used to configure the individual jobs that will be executed in the pipeline.
Each job is defined by a name, and a set of commands that will be executed in the job.
The commands are executed in the order that they are defined in the job.

## Job configuration

Just like it is possible to define the image in the `global` section, it is also possible to define configuration for each job in the `jobs` section.
This is useful if you want to use a different image for each job, or if you want to use a different set of environment variables for each job.

When the image is defined in the `jobs` section, it will override the image defined in the `global` section.
On top of this intricacy, it is also to possible to specify the configuration per command.

Note that the configuration defined per job is optional. If the configuration is not defined, the configuration defined in the `global` section will be used.

For more information on you can take a look at the [specification](specification.md).

Let's take a look at the following example in order to break it down and figure out what is happening:
For this example let's take a look at the `jobs` section of our example:

```yaml
...
  # Node V18
  - name: setup-node_18
    image: registry.hub.docker.com/library/node:18
    environment:
      APP_ENV: production
      APP_DEBUG: false
    commands:
     - npm ci
    artifacts:
      - name: node_modules_18
        path: ./node_modules/

  - name: generate-docs_18
    commands:
     - npm run build
    resources:
      - node_modules_18
    artifacts:
      - name: dist_folder
        path: ./dist/
...
```

- `name`: the name of the job. this is used to identify the job in the nosana network.
- `commands`: the shell commands that will be executed in the job.
- `artifacts`: the artifacts that will be generated by the job.
  - `name`: the name of the artifact.
  - `path`: the path to the artifact.
- `resources`: the resources that will be available to the job.
  - `name`: the name of the resource.
  - `path`: the path to the resource.

In the example above we have defined 2 jobs: `setup-node` and `generate-docs`.
The `setup-node` job will install the dependencies for the project, and the `generate-docs` job will generate the documentation site.
The `setup-node` job will generate an artifact called `node_modules`, and the `generate-docs` job will generate an artifact called `dist_folder`.
The `generate-docs` job will also use the `node_modules` artifact that was generated by the `setup-node` job.

So it is important to keep in mind that the artifacts that are generated by one job can be used by another job. this is useful if you want to share data between jobs.
Eventually this is useful to produce an artifact that will deployed to production.
in this case that would be the `dist_folder` artifact, which contains a static HTML with some as

:::
