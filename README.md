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
