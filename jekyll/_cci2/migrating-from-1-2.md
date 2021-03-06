---
layout: classic-docs
title: "Migrating from 1.0 to 2.0"
short-title: "Migrating from 1.0 to 2.0"
description: "Why and how to migrate from CircleCI 1.0 to 2.0"
categories: [migration]
order: 15
---

CircleCI 2.0 introduces the requirement that you create a configuration file (`.circleci/config.yml`), and it adds new required keys for which values must be defined. This release also allows you to use multiple jobs in your configuration. **Note:** If you configure multiple jobs, it is important to have parallelism set to `1` to prevent duplication of job runs.

If you already have a `circle.yml` file, this article will help you make a copy your existing file, create the new required keys, and then search and replace your 1.0 keys with 2.0 keys. If you do not have a `circle.yml` file, refer to the [Sample 2.0 `config.yml` File]({{ site.baseurl }}/2.0/sample-config) to get started from scratch.

* Contents
{:toc}

## Steps to Configure Required 2.0 Keys

1. Copy your existing `circle.yml` file into a new directory called `.circleci` at the root of your project repository.

2. Rename `.circleci/circle.yml` to `.circleci/config.yml`.

3. Add `version: 2` to the top of the `.circleci/config.yml` file.

4. Add the following two lines to your `config.yml` file, after the version line. If your configuration includes `machine:`, replace `machine:` with the following two lines, nesting all of the following sections under `build`.
     ```
     jobs:
       build:
     ```
5. Add the language and version to your configuration using either the `docker:` and `- image:` keys in the example or by setting `machine: true`. If your configuration includes language and version as shown for `ruby:` below, replace it as shown.
     ```
       ruby:
         version: 2.3
     ```
     Replace with the following two lines:
     ```
         docker:
           - image: circleci/ruby:2.3
     ```
     The primary container is an instance of the first list image listed. Your build commands run in this container and must be declared for each job.

6. The `checkout:` step is required to run jobs on your source files. Nest `checkout:` under `steps:` for every job by search and replacing
     ```
     checkout:
       post:
     ```
     With the following:
     ```
         steps:
           - checkout
           - run:
     ```
7. Validate your YAML at <http://codebeautify.org/yaml-validator> to check the changes. 

## Steps to Configure Workflows

Optionally configure workflows, using the following instructions:

1. To use the Workflows feature for job orchestration, first split your build job into multiple jobs, each with a unique name.
 
2. As a best practice, add lines for `workflows:`, `version: 2` and `<workflow_name>` at the *end* of the master `.circle/config.yml` file, replacing `<workflow_name>` with a unique name for your workflow. **Note:** The Workflows section of the `config.yml` file is not nested in the config. It is best to put the Workflows at the end of the file because the Workflows `version: 2` is in addition to the `version:` key at the top of the `config.yml` file.  
     ```
     workflows:
       version: 2
       <workflow_name>:
     ```  
3. Add a line for the `jobs:` key under `<workflow_name>` and add a list of all of the job names you want to orchestrate. In this example, `build` and `test` will run in parallel.
 
     ```
     workflows:
       version: 2
       <workflow_name>:
           jobs:
             - build
             - test
     ```  
4. For jobs which must run sequentially depending on success of another job, add the `requires:` key with a nested list of jobs that must succeed for it to start. If you were using a `curl` command to start a job, Workflows enable you to remove the command and start the job by using the `requires:` key.
 
     ```
      - <job_name>:
          requires:
            - <job_name>
     ```
5. For jobs which must run on a particular branch, add the `filters:` key with a nested `branches` and `only` key. For jobs which must not run on a particular branch, add the `filters:` key with a nested `branches` and `ignore` key. **Note:** Workflows will ignore job-level branching, so if you have configured job-level branching and then add workflows, you must remove the branching at the job level and instead declare it in the workflows section of your `config.yml`, as follows:
 
     ```
     - <job_name>:
         filters:
           branches:
             only: master
     - <job_name>:
         filters:
           branches:
             ignore: master
     ```     
6. Validate your YAML again at <http://codebeautify.org/yaml-validator> to check the changes.

## Search and Replace Deprecated 2.0 Keys

- If your configuration sets a timezone, search and replace `timezone: America/Los_Angeles` with the following two lines:

```
    environment:
      TZ: "/usr/share/zoneinfo/America/Los_Angeles"
```

- If your configuration modifies $PATH, add the path to your `.bashrc` file and replace 

```
    environment:
    PATH: "/path/to/foo/bin:$PATH"
```

With the following to load it into your shell (the file $BASH_ENV already exists and has a random name in /tmp):

```
    steps:
      run: echo 'export PATH=/path/to/foo/bin:$PATH' >> $BASH_ENV 
      run: some_program_inside_bin
```

- Search and replace the `hosts:` key, for example:

```  
hosts:
    circlehost: 127.0.0.1
```

With an appropriate `run` Step, for example:

```
    steps:
      - run: echo 127.0.0.1 circlehost | sudo tee -a /etc/hosts
```


- Search and replace the `dependencies:`, `database`, or `test` and `override:` lines, for example:

```
dependencies:
  override:
    - <installed-dependency>
```

Is replaced with:

```
      - run:
          name: <name>
          command: <installed-dependency>
```

- Search and replace the `cache_directories:` key:

```
  cache_directories:
    - "vendor/bundle"
```

With the following, nested under `steps:` and customizing for your application as appropriate:

```
     - save_cache:
        key: dependency-cache
        paths:
          - vendor/bundle
```

- Add a `restore_cache:` key nested under `steps:`.

- Search and replace `deployment:` with the following, nested under `steps`:

```
     - deploy:
```

**Notes on Deployment:**

- See the [Writing Jobs with Steps]({{ site.baseurl }}/2.0/configuration-reference/#deploy) document for valid `deploy` options to configure deployments on CircleCI 2.0
- Please read the [Deployment Integrations]({{ site.baseurl }}/2.0/deployment_integrations/) doc for examples of deployment integration for CircleCI 2.0.

## Validate YAML

When you have all the sections in `.circleci/config.yml` we recommend that you validate your YAML syntax using a tool such as <http://codebeautify.org/yaml-validator>. Fix up any issues and commit the updated `.circleci/config.yml` file. When you push a commit the job will start automatically and you can monitor it in the CircleCI app.
