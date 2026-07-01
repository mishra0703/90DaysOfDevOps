# GitHub-Hosted vs Self-Hosted

| | GitHub-Hosted | Self-Hosted |
|---|---|---|
| Who manages it? | GitHub Itself | Us (who created it) |
| Cost | Free for public repos & limited free minutes for private repos (then paid) | Our own infra cost (EC2, server, electricity) — can be cheaper at scale or more expensive depending on usage |
| Pre-installed tools | Yes , Many softwares comes pre-installed | We have to install whatever we need — full control but more setup work |
| Good for | Open source projects, small-medium teams, standard pipelines where default tools are enough | Large teams with heavy workloads, pipelines needing custom hardware/software (e.g. GPU builds, specific OS, private network access, our own EC2) |
| Security concern | Code runs on GitHub's shared infrastructure — fine for most, but sensitive/proprietary code runs on someone else's machine | Full control over the environment — but we're responsible for keeping the runner secure, patched, and isolated |


*Note :*
self-hosted runners are exactly what we'd use if our pipeline needs to SSH into a private EC2, access internal services, or use credentials that can't leave our network — which maps directly to our deploy.yml setup where the runner needs private EC2 access.


