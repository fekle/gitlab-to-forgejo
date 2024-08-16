# GitLab to Forgejo migration tool

This is a simple tool to quickly migrate all your GitLab organizations to Forgejo.

Currently, it supports migrating users, organizations (groups on GitLab), and repositories (projects on GitLab).

**It does not migrate group and project memberships or permissions, as that is beyond the scope of what I needed.**

Another thing to keep in mind is that GitLab limits the REST API page size to 100. That is enough for my usage, as I don't have more than 100 groups or users, and no groups or users with more than 100 projects. So, because I'm lazy, I didn't implement pagination.

### Users

Migrated users will be created with a random password and will need to reset it on first login.
Also, for security reasons, migrated users will always be prohibited from logging in and need to be enabled by an admin.

External users in GitLab will be migrated as restricted users in Forgejo.
If a user has a status of `active` in GitLab, they will be created as active in forgejo. Remember that `prohibit_login` will always be set to true for migrated users.
If a user is allowed to create a group in GitLab, they will be allowed to create an organization in Forgejo.
If a user is allowed to create projects in GitLab, the maximum number of projects they can create in Forgejo will be set to the Forgejo default of `-1`, otherwise it will be set to `0`.

**Existing users on Forgejo will not be modified.**

### Visibility

Visibility for projects, groups, and users will be mapped from GitLab to Forgejo as follows:

- `public` -> `public`
- `internal` -> `internal`
- `private` -> `private`

For Forgejo entities where there is only a `private` boolean property, it will be set to true when the GitLab visibility is `private`.

### Project Migration

This tool uses Forgejo's server-side repository migration, the same that you can manually do from the web interface.
Therefore, it should be quite fast and reliable and supports all features Forgejo supports. All migration features are enabled.
Projects will not be configured as a mirror.

## Usage

To run this script, you will need Elixir. I've tested it with Elixir 1.17, but it should work with other recent versions.

I have tested this tool with GitLab `17.2.2-ce` and Forgejo `8.0.1`.

First, create a config file called `config.exs` in the root of the project. It should look like this:

```elixir
import Config

config :gitlab_to_forgejo,
  forgejo_host: "https://forgejo.example.com",
  forgejo_token: "forgejo-token",
  gitlab_host: "https://gitlab.example.com",
  gitlab_token: "gitlab-token",
  delete_projects: false

```

The properties `gitlab_host` and `forgejo_host` should be self-explanatory.
For both tokens, you need to create a personal access token in the respective web interface. I created tokens with full access permissions and deleted them after the migration.

When `delete_projects` is set to `false`, this script will skip any projects that already exist in forgejo; otherwise, it will delete them before starting the migration. Consider this option dangerous.

Then, just run the script with `elixir gitlab_to_forgejo.exs`.

Enjoy!
