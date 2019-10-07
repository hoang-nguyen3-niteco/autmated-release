# Automated Release

This is POC for automated release through Jenkins pipeline 

## Getting started

- Your Jenkins instance with `publicIp` was exposed
- Creating a new repo in Jenkins  (Multi Branch Pipeline) and setting SCM to your Git repository
- Setting hook as `http(s)://<JENKINS_PUBLIC_IP>:<JENKINS_PORT>/github-webhook` in your Github repository

## How it work

Below is the flow of automated release on Jenkins.This flow is defined at [Jenkinsfile](./Jenkinsfile)

- If any change is detected from branch whose name match with the regrex pattern `release/*`, it will ask the user input to release new version
- If input version is valid, the step `bump version` will be processed. The version in `package.json` and the value of `IMAGE_VERSION` will be edited as input version. Then, the new commit will be added and push to Git
- After `bump version` stage is success, the next step is `Creating PR`. In this stage, Github API is used to create Pull Request (by curl). The result of creating PR is verified by created pull request number.
- If no created pull request number was found and `bump version` stage is success, it 'll jump to `revert` because no PR was created so we need to revert commit we created in the `bump version` stage.
- If `pull_request_number` was found, the `merge PR` stage will be triggered. We still leverage Github API to merge pull request and verify the result based on HTTP response status of merging API. HTTP response status equals to 200, it mean success, else it fail
- If merging is success, next `tagging` will be processed. The tag will be pointed at the merge commit. Tag name will be `input version` then it will be pushed
- If `tagging` stage finished without error, the last work is syncing `develop` branch with master. Sync it with master then push it into origin remote