# New Leaf Expansion App Docs

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

To access the databases in production or staging, you will need to port forward into a pod. The `db-gateway` pod in the production environment was created for this purpose. Run `kubectl port-forward pod/db-gateway 8080:5432 -n new-leaf`, then you'll be able to access

## CI pipeline
This involves manually triggering GitHub workflows at certain points.
- When you push to the frontend or backend repos, the `Staging deploy` workflow is automatically triggered which deploys to staging
- After the workflow completes, go to the [E2E repo](https://github.com/for-social-change/new-leaf-e2e) and manually run the `e2e tests` workflow which will run the Cypress E2E tests on staging
- If the workflow completes without errors or test failures, go back to the FE/BE repo and manually run the `Production deploy` workflow to deploy to production

## Services we're paying for
- Amazon Web Services - app infrastructure
- Twilio - for sending SMS
- SendGrid - for sending verification and password recovery emails (100 emails/day is free)
- Google Domains - for fschome.org and fschome-staging.org
