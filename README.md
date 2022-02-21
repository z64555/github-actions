# github-actions
A repo for testing github-actions

This is just an experimental repo for messing around with the Github actions feature, so expect things to be broken.

I won't support any forks based on this project until further notice.

I may keep findings within this readme, however.

# Overview
"Workflows" are Github's version of other Ci's "Pipelines", they are broken down further into "Jobs" and "Steps."
Jobs are run on Github's servers, and each individual job has its own Virtual Machine instance. Jobs that depend on
other jobs are run in sequence, and different jobs can be run in parellel.  In addition, should a job have a "matrix"
specified, that job can run in parellel with different configurations specified within the matrix.

Since jobs run in their on VM, and that VM can be of a different archetecture, passing data and files between job
instances shouldn't rely on files held in the filesystem to persist between jobs.  Instead, Artifacts and Caches are
used.

Jobs have "Steps" within them which can either call another workflow (called a reusable workflow), an Action, or run a
command on a number of supported command line shells.  Actions can be seen as a type of workflow with one job and one
step, but they have some other restrictions to them such as Environment variables not being retained (you have to pass
them into the Action as an input).  Github has a marketplace to look for actions which do a variety of useful things.

Unfortunately, in order actually do anything you are basically reduced to using an Action or running something on the
command line, which is usually a .sh script or another commandline program.

# Triggers
Workflows run whenever they are kicked off by a trigger, which is declared within the individual workflow .yaml within
the `on:` object (borrowing a bit of language from JSON spec).  There's a wide list of triggers the most notable are:
	push:			Triggered on a new commit
	pull_request:	Triggered on new pull request
	label:			Triggered on new label
	schedule:		Triggered on a timer
	workflow_dispatch:	Triggered manually from the github website. Note: Can only be triggered if the .yaml is in the main/master branch
	workflow_run:		Triggered when another workflow runs or completes
	workflow_call:		Triggered from another workflow, making this workflow a "reuable" workflow

# Manual Triggers
For some reason the runners refuse to pick up the test workflows that aren't the example main.yml.  Even when trying to
edit the main.yml to have multiple jobs in them runners still refuse it.  Not sure if this is a billing limitation or
some other quirk

Before I forget, manual triggers can request inputs from the github webpage.  It doesn't document the types very well,
but they are: `string`, `boolean`, `choice`, and `environment`
`string`:	Default type. Displays a input textbox on the trigger submission form.
`boolean`:	Displays a checkbox on the trigger submission form
`choice`:	Either displays a radio button group or a dropdown list box to select an option.
`environment`:	No idea if this is the run-time environment or something else.

example:
```
on:
  workflow_dispatch:	# Trigger event, raised by submission form on github website.  Only works if the .yaml is in the master branch
    inputs:			# github.event.inputs
      logLevel:		# github.event.inputs.logLevel
        description: 'Log level'	# Description/label shown on the form
        required: true				# bool; If it is required, is marked with an * on the form
        default: 'warning' 			# default value to use
        type: choice				# input data type; string, boolean, choice, or environment
        options:					# choice type has .options
        - info
        - warning
        - debug 
      tags:			# github.event.inputs.tags
        description: 'Test scenario tags'
        required: false 
        type: boolean
      environment:
        description: 'Environment to run tests against'
        type: environment
        required: true
```

# Notes on Artifacts
  Artifacts are files that jobs can upload to github and are visible to all other jobs and actions within the workflow.
  They persist after a workflow has been run, and may be re-usable during a re-run of the workflow.  Issue is detecting
if they exist first.
  Artifacts can have a customizable rentention rate, for public repo's this is anywhere from 1 to 90 days maximum.  This
policy may be changed at the repo side, and it my also be changed at the action side.  If set by an action, the
rention period may only be equal or shorter than the repo's settings.
  There does not appear to be a size limit for artifacts.

# Notes on Caches
  Caches are similar to artifacts but persist in a different domain on github and are not accessible through the web
browser like artifacts can be after a workflow finishes.  They do however persist across individual workflow runs.
  Github has a hard limit of 10GB worth of caches, and starts evicting cached data should the total cache footprint
exceeds 10GB or if the cache hasn't been touched within a week.  They are recommended for use for dependencies that are
frequently used across workflows (Such as the Qt libraries).
