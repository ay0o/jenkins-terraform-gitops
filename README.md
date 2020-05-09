Jenkins pipeline to implement GitOps for a Terraform repository in a self-hosted GitLab.

On merge request, tests (`fmt` and `validate`) are run.
On push to master, code is applied right away.

That's it, simple but effective.