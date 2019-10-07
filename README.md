# Automated Release

This is POC for automated release through Jenkins pipeline 

## Getting started

- Creating a new repo in Jenkins  (Pipeline) and setting SCM to match your Git repository
- To run pipeline, click on ` Build with Parameters`, there are two parameters need to be filled in here
        
    - PROJECT_URL: URL of Git repository in HTTPS protocol. For example: `https://github.com/hoang-nguyen3-niteco/app-fake`
    - BRANCH: the branch is used to as base branch to process release

## How it work

Below is the flow of automated release on Jenkins.This flow is defined at [Jenkinsfile](./Jenkinsfile)

- First, it will ask the user input to release new version. If input version is valid, the step `bump version` will be processed. The version in `package.json` and the value of `IMAGE_VERSION` will be edited as input version. Then, the new commit will be added and push to Git. Otherwise, the process will be aborted.
- After `bump version` stage is success, the next step is `Creating PR`. In this stage, Github API is used to create Pull Request (by curl). The result of creating PR is verified by created pull request number which is found from response body.
- If no created pull request number was found and `bump version` stage is success, it 'll jump to `revert` because no PR was created so we need to revert commit we created in the `bump version` stage.
- If `pull_request_number` was found, the `merge PR` stage will be triggered. We still leverage Github API to merge pull request and verify the result based on HTTP response status of merging API. HTTP response status equals to 200, it mean success, else it fail
- If merging is success, next `tagging` will be processed. The tag will be pointed at the merge commit. Tag name will be `input version` then it will be pushed
- If `tagging` stage finished without error, the last work is syncing `develop` branch with master. Sync it with master then push it into origin remote