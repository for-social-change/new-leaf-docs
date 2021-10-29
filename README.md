# New Leaf Expansion App

## Things to do before project launch
- Create the client info CSV import script
- Load testing
- Delete test accounts on production (see [database section](https://github.com/for-social-change/new-leaf-docs/#database-access) and [Test accounts section](https://github.com/for-social-change/new-leaf-docs/#test-accounts))
- There are some features that aren't covered by automated tests and I checked that they worked, but you may want to double-check these closer to the project launch:
  - Password recovery
  - Creating a new account
  - Cronjob for sending SMS for upcoming appointments and surveys that aren't completed (`cronjob-reminders`)
  - Cronjob for resetting full resources (`cronjob-resources`)

## Links
- Production - https://fschome.org
- Staging - https://fschome-staging.org
- [Backend architecture and account creation flow diagram](https://app.diagrams.net/#G1kXaIACwYXgDjok23fmnl3792eVtrb1vK) (also in this repo as a pdf)
- [Google doc with user requirements and future features](https://docs.google.com/document/d/13GYdpAW-aUbapH_Jkd6RKOgT9wqpYQ2f9lxtoPk7MZo/edit)

### Repositories
- [Infrastructure](https://github.com/for-social-change/new-leaf-infrastructure)
- [Frontend](https://github.com/for-social-change/new-leaf-frontend)
- [Backend](https://github.com/for-social-change/new-leaf-backend)
- [E2E tests](https://github.com/for-social-change/new-leaf-e2e)

## Background
This app's infrastructure was set up using a bootstrapping tool called [Zero](https://github.com/commitdev/zero) in July 2021. Read the [docs here](https://getzero.dev/docs/zero/about/overview) for more information and [join the Slack group](https://slack.getzero.dev/) to ask the team if you have any questions.

Our configuration uses:
- GitHub Actions for CI
- [zero-notification-service](https://github.com/commitdev/zero-notification-service) for sending SMS using Twilio and emails using SendGrid
- Ory Kratos and Oathkeeper for user management and authorization
- NodeJS backend with Postgres database
- React frontend and no landing page
- No logging to save on costs

## Local Development
Although [using Teleprescence](https://getzero.dev/docs/modules/aws-eks-stack/guides/dev-experience-telepresence) to run the app locally using the staging environment database is enabled, I found that simply running it using Docker was easier since this is a relatively simple app with just one backend service.

### FE and BE
Here are the steps to getting the app running locally:
1. Clone the [frontend](https://github.com/for-social-change/new-leaf-frontend) and [backend](https://github.com/for-social-change/new-leaf-backend) repos.
2. In the BE repo, run `npm install` to install packages, then (after making sure you're running Docker) run `docker-compose -f local-auth-setup.yml up` which will run the BE locally and do the database migrations.
3. In the FE repo, run `yarn` to install packages, then run `yarn start` to run the FE on http://127.0.0.1:3000/.

To delete your local databases, run `docker-compose -f local-auth-setup.yml down -v`.

### Infrastructure
The `admin-user` IAM user should used for updating the infrastructure with Terraform and AWS CLI. You'll need to [set up your AWS credentials](https://docs.aws.amazon.com/polly/latest/dg/setup-aws-cli.html) with the `admin-user` access key and secret key. Log into AWS with the root user `admin@forsocialchange.org` and create a new access key for `admin-user` to use as your AWS credentials. 

To update the infrastructure, clone the [infrastructure repo](https://github.com/for-social-change/new-leaf-infrastructure), make your updates, and run `make apply`.

Debugging infrastructure issues was mostly done using `kubectl` and looking at the pods' logs, etc. 

The `zero-project-safe.yml` file in the [infrastructure repo](https://github.com/for-social-change/new-leaf-infrastructure) is the seed file that Zero used to set up the infrastructure. The AWS access keys and github access tokens were removed before pushing to the repo.

### Database access
There are 2 databases:
- The `userauth` database contains the `identities` (objects containing `email` and `user_type`) and related tables that are managed by Kratos.
- The `newleaf` database contains the rest of the data for the app. The `auth_id` field of a row in the `users` table is the id of the corresponding user in the `identities` table in the `userauth` database. You can learn more about how the users and accounts are created on the right side of [this diagram](https://github.com/for-social-change/new-leaf-docs/blob/main/BackendArchitectureAndAccountCreationFlows.pdf).

To access the databases during local development with Docker, run the BE and then open a postgres client like Postico. 

Fill in the fields like this to access the `newleaf` database: 
- Host: 127.0.0.1
- Port: 5432
- User: foo
- Password: foofoofoo
- Database: new-leaf

And fill in the fields like this to access the `userauth` database: 
- Host: 127.0.0.1
- Port: 5433
- User: kratos
- Password: secret
- Database: kratos

These fields are all found and can be configured in `local-auth-setup.yml` in the BE repo.

To access the databases in production or staging, you will need to port forward into a pod. The `db-gateway` pod in the production environment was created for this purpose (there isn't one in staging though, but you can create it by referencing [this handy post](https://github.com/kubernetes/kubernetes/issues/72597#issuecomment-518617501). Make sure you are in the production context in kubectl with ` kubectl config use-context new-leaf-prod-ca-central-1` and run `kubectl port-forward pod/db-gateway 8080:5432 -n new-leaf`. 

Now you can set up a profile on Postico with these fields for both databases:
Host: 127.0.0.1
Port: 8080
Database: `userauth` or `newleaf`

The User and Password for both databases can be found in the `new-leaf/kubernetes/prod/new-leaf` secret in AWS Secrets Manager.

## CI pipeline
This involves manually triggering GitHub workflows at certain points.
- When you push to the frontend or backend repos, the `Staging deploy` workflow is automatically triggered which deploys to staging
- After the workflow completes, go to the [E2E repo](https://github.com/for-social-change/new-leaf-e2e) and manually run the `e2e tests` workflow which will run the Cypress E2E tests on staging. Avoid making changes on staging because they might cause these tests to fail.
- If the workflow completes without errors or test failures, go back to the FE/BE repo and manually run the `Production deploy` workflow to deploy to production

## Test accounts
The test accounts use maildrop.cc emails for convenience but they are not secure (anyone can access these inboxes, go see at maildrop.cc) so the production test accounts should be deleted before the project launches. To delete accounts, delete the applicable rows in the `users` and `identities` tables (see the [Database Access section](https://github.com/for-social-change/new-leaf-docs/#database-access) for more info).

Staging
- Researcher - fsc.researcher1@maildrop.cc
- Connector - fsc.connector1@maildrop.cc
- Another connector - fsc.connector2@maildrop.cc
- Cash client - fsc.client.cash1@maildrop.cc
- Non-cash client - fsc.client.cash2@maildrop.cc

Production
- Researcher - fsc.researcher1@maildrop.cc
- Connector - robyn@forsocialchange.org
- Cash client - fsc.client1@maildrop.cc
- Another cash client - fsc.client2@maildrop.cc
- Non-cash client - fsc.client3@maildrop.cc

## Services we're paying for
- Amazon Web Services - app infrastructure
- Twilio - for sending SMS
- SendGrid - for sending verification and password recovery emails (100 emails/day is free)
- Google Domains - for fschome.org and fschome-staging.org
