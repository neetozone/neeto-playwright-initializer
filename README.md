# neeto-playwright-initializer
Initialize Playwright tests with just one command.

## Usage

This repository is used to initialize a Playwright test directory for a new neeto app. Executing the actions in this repository will let you
create a Playwright test with setup, teardown and all necessary configurations needed for the neeto ecosystem by simply answering a few questions.

### Pre-requisites

1. Create a new project in [neetoPlaydash](https://neeto-engineering.neetoplaydash.com) for the neeto product in which you want to run the tests in.
2. Create a new project in [Currents.dev](https://app.currents.dev) for the neeto product in which you want to run the tests in.

### Steps

1. Clone the repository and install all the dependencies by executing the command: `yarn install`.
2. Create the Playwright application by executing the command: `yarn generate`.
3. You will be asked a series of questions for which you have to provide the proper answers.
4. Once done a new directory called `playwright-tests-starter` will be generated.
5. Copy this directory within the neeto app for which you want to write the tests for and rename it to `playwright-tests`.
6. Navigate inside the directory and initialize it by executing `yarn install`.
7. Execute `yarn playwright:headed` and the tests should run properly.

## Steps after initialization

### Adding CI configuration

You need to add the neeto-ci configurations for executing the Playwright tests in the review environment and staging environment and you have to configure
the nightly runs. For this create three new files in the .neetoci directory:

```bash
# Execute at the root of the neeto app

touch .neetoci/playwright-nightly-staging.yml
touch .neetoci/playwright-staging.yml
touch .neetoci/playwright.yml
```

Now add the contents of the newly created configurations as given below. Replace all occurrences of `<neetoapp>` with the proper app name.
Eg: `neetoform`, `neetocal`

```yml
# .neetoci/playwright-nightly-staging.yml

version: v1.0

fail_fast: false
is_cypress: true
parallelism: 4
plan: professional-2

envs:
  - key: TEST_ENV
    value: staging
  - key: DEPLOYMENT_URL
    value: http://www.neeto.com/<neetoapp>?env=staging

global_job_config:
  setup:
    - checkout
    - neetoci-version node 18.12
    - cache restore
    - yarn install
    - cd playwright-tests
    - yarn install
    - yarn playwright install --with-deps
    - cache store
  jobs:
    - name: Playwright Staging Nightly
      commands:
        - yarn playwright:ci --shard=$((${JOB_COMPLETION_INDEX}+1))/4

triggers:
  - event: cron
    cron_line: "0 3,15 * * MON,TUE,WED,THU,FRI"
    branch: main

```

```yml
# .nnetoci/playwright-staging.yml

version: v1.0

fail_fast: false
is_cypress: true
parallelism: 4
plan: professional-2

envs:
  - key: TEST_ENV
    value: staging
  - key: DEPLOYMENT_URL
    value: http://www.neeto.com/<neetapp>?env=staging

global_job_config:
  setup:
    - checkout
    - neetoci-version node 18.12
    - cache restore
    - yarn install
    - cd playwright-tests
    - yarn install
    - yarn playwright install --with-deps
    - cache store
  jobs:
    - name: Playwright Staging
      commands:
        - yarn playwright:ci --shard=$((${JOB_COMPLETION_INDEX}+1))/4

lifecycle:
  on_running:
    - action: remove_labels
      labels: ["playwright-staging-completed", "playwright-staging-run"]
    - action: add_labels
      labels: ["playwright-staging-triggered"]

  on_complete:
    - action: remove_labels
      labels: ["playwright-staging-run", "playwright-staging-triggered"]
    - action: add_labels
      labels: ["playwright-staging-completed"]

triggers:
  - event: pull_request_labeled
    label: playwright-staging-run

```

```yml
# .nnetoci/playwright.yml

version: v1.0

fail_fast: false
is_cypress: true
parallelism: 4
plan: professional-2

envs:
  - key: TEST_ENV
    value: review

global_job_config:
  setup:
    - checkout
    - neetoci-version node 18.12
    - cache restore
    - yarn install
    - cd playwright-tests
    - yarn install
    - yarn playwright install --with-deps
    - cache store
  jobs:
    - name: Playwright Review
      commands:
        - yarn playwright:ci --shard=$((${JOB_COMPLETION_INDEX}+1))/4

lifecycle:
  on_running:
    - action: remove_labels
      labels: ["playwright-completed", "playwright-run"]
    - action: add_labels
      labels: ["playwright-triggered"]

  on_complete:
    - action: remove_labels
      labels: ["playwright-run", "playwright-triggered"]
    - action: add_labels
      labels: ["playwright-completed"]

triggers:
  - event: pull_request_labeled
    label: playwright-run

```

### Moving the generated .gitignore entries

When generating the starter Playwright code, we generate some required .gitignore files along with it. Since the application already has a
.gitignore file, its redundant to keep two files. So we can move all the .gitignore entries in the `playwright-tests` directory to the one
in the root of the application. To do this you can copy the entries in the the `playwright-tests/.gitignore` file to the `.gitignore` file
and prefix all the of them with `/playwright-tests/`

```
# playwright-tests/.gitignore

node_modules/
/test-results/
/playwright-report/
/blob-report/

...
```

```
# .gitignore

/playwright-tests/node_modules/
/playwright-tests/test-results/
/playwright-tests/playwright-report/
/playwright-tests/blob-report/

...
```
