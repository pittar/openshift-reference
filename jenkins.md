# Jenkinsfiles

## Git Credentials

Reference git credentials (linked to the `jenkins` service account) when cloning in a Jenkinsfile.

```
git url: `ssh://git@my.company.com/repo.git',  branch: 'master', credentialsId: 'git-secret-name'
```
