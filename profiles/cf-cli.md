# Manage Cloudflare from a project checkout

Run commands from the project root. Use the repository’s package scripts for development, release and smoke checks. Use the Cloudflare CLI (`cf`) for account and resource operations. Read the release script when both cf and Wrangler configs exist; the script selects the build, environment and verification gates.

## Select the project and account

Use the repository’s locked CLI when present. In the commands below, substitute `pnpm exec cf` or `npm exec -- cf` after installing the locked dependencies. Otherwise use the official Cloudflare CLI installed on the machine. Check `cf --version` to identify the executable.

On a developer machine, choose the profile for the account that owns the project. For example, in a checkout whose documented profile is `personal`:

```sh
cf auth list
# Once per machine, if the named profile is missing:
cf auth create personal
# Once per checkout, including a new worktree:
cf auth activate personal "$(pwd -P)"
cf auth whoami
```

Activation binds that directory and its children. Keep the binding at the checkout root so sibling projects can select other accounts. Run `whoami` from the same environment and directory as the intended command. Confirm the account and target Worker, zone or storage resource against the runbook for the selected environment.

Credential and account selection are independent. A `CLOUDFLARE_API_TOKEN` (including one loaded from a dotenv file) takes precedence over profiles. Next come the explicit profile flag, `CLOUDFLARE_AUTH_PROFILE`, the nearest directory binding, and the default login. `CLOUDFLARE_ACCOUNT_ID` takes precedence over the project's cf config. For unattended work inject a token scoped to the needed account, resources and operations through the job's secret store. Keep credential values out of logs and version control. Wrangler selects credentials separately. [Cloudflare authentication and account selection](https://developers.cloudflare.com/cf/get-started/)

## Develop, release and operate

| Task | Project workflow |
| --- | --- |
| Develop | Run the local dev script. For a new cf project use `cf dev`. For local storage operations look up support for `--local` on that specific command. |
| Build or preview | Use the repo's build and deploy-check scripts. For a cf-native project without wrappers, use `cf build` and `cf deploy --dry-run`. Inspect the resulting Worker name, mode, bindings, triggers and domains. |
| Deploy | Run the documented release command. Preserve its migrations, backup, build, verification receipt and smoke steps. If the project ships directly with cf, deploy the selected mode and verify the serving version and affected behavior. |
| Operate a resource | Search by action and resource type, inspect arguments and request shape, then select the resource ID for the project and mode. Preview mutations where supported and read back the resulting state. |
| Diagnose or recover | Start with the deployed version and a request or job ID. Use the project's log or trace read path and its rollback procedure. Code rollback and data recovery are separate operations. [Cloudflare operations](../guides/cloudflare.md) |

Example of discovering a D1 operation without resource names or credentials:

```sh
cf cli search "list database migrations"
cf d1 migrations list --help
```

For generated API operations, `cf schema <command path>` shows the request shape. Check pagination on list commands. Record the target, version or resource ID, operation and verification result without secret values. [CLI discovery](https://developers.cloudflare.com/cf/agents/)

## Choose the project tooling

For a new Workers project, start with cf and a typed `cloudflare.config.ts`. In a Wrangler-only project, use cf for resource operations and the existing Wrangler scripts for local builds and releases. Running cf project commands against that checkout can introduce a second config. Treat a migration as a separate change: preview `cf migrate --dry-run`, resolve generated TODOs and compare bindings, domains, triggers, assets, migrations and staging behavior before switching the release path. [Migration guide](https://developers.cloudflare.com/cf/wrangler/migrate/)

In each project's instructions, record its CLI invocation, account/profile selection, config per environment, local dev command, release preview, deployment and post-deploy check. Link the existing runbook when it already supplies those details.
