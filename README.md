# CI/CD Playground

Working deployment examples for [DEP 9000](https://github.com/Black-HOST/deployer), the GitHub Action and Docker image that deploys a repository to any server over FTP, SFTP or rsync.

Each example lives on its own branch, deploys itself on every push and is live on [cicd.guide](https://cicd.guide/). What you see there is what the pipeline produced.

## Examples

| Branch | Protocol | Auth | Deploys | Live | Guide |
|---|---|---|---|---|---|
| [`ftp`](https://github.com/Black-HOST/CI-CD/tree/ftp) | FTP (FTPS) | password | static site to cPanel | [cicd.guide/ftp/](https://cicd.guide/ftp/) | in progress |
| [`sftp`](https://github.com/Black-HOST/CI-CD/tree/sftp) | SFTP | SSH key | static site to cPanel | [cicd.guide/sftp/](https://cicd.guide/sftp/) | in progress |

Coming next: `rsync`, `laravel`, `node`.

## Use an example

Every branch README has the same four steps: get the files, set the branch that deploys, add the secrets, push.

For a new project, take the branch as a starting point:

```sh
git clone -b ftp --single-branch https://github.com/Black-HOST/CI-CD.git my-site
```

For an existing project, copy the branch's `.github/workflows/deploy.yml` into it.

## Let your agent do it

Every branch README ends with a prompt for Claude Code, Cursor and similar tools. The agent writes the pipeline; the credentials go into GitHub secrets, never into the chat or the repository.

## Links

- [DEP 9000](https://github.com/Black-HOST/deployer): the action, every option, GitLab CI quick start
- [`blackhost/deployer`](https://hub.docker.com/r/blackhost/deployer) on Docker Hub
- [BlackHOST Help Center](https://black.host/hc/): the guides behind these examples
- Issues with an example go here; issues with the action go to the [deployer repository](https://github.com/Black-HOST/deployer/issues)
