# Container Image Build with scanning quickstart

This repository shows an example of building a container image while leveraging Google Cloud's artifact scanning for docker images.

## Pre-requisites
* A Google Cloud Project
* gcloud cli installed
* A github account
* The following apis enabled in the GCP project: cloudbuild, containerregistry, ondemandscanning, and artifactregistry

```bash
gcloud services enable cloudbuild.googleapis.com \ 
containerregistry.googleapis.com \
ondemandscanning.googleapis.com \
artifactregistry.googleapis.com \
```
* Grant the cloudbuild service account access to run on-demand scanning:
```bash
gcloud projects add-iam-policy-binding <project_id> \
    --member="serviceAccount:<project_number>@cloudbuild.gserviceaccount.com" \
    --role="roles/ondemandscanning.admin"
```

* fork the following repository

* 

* Install the google cloudbuild github app: https://cloud.google.com/build/docs/automating-builds/build-repos-from-github#installing_gcb_app, skip the add trigger part

* create a cloudbuild trigger from the repository <b>Note</b> please replace REPOSITORYOWNER with your github username:
```
gcloud beta builds triggers create github \
    --name=test-image \
    --repo-name=pso-infra-cicd-docker-scan-demo \
    --repo-owner=REPOSITORYOWNER \
    --pull-request-pattern=^main$ \
    --build-config=cloudbuild_test.yaml \
    --comment-control=COMMENTS_ENABLED_FOR_EXTERNAL_CONTRIBUTORS_ONLY
```
```
gcloud beta builds triggers create github \
    --name=deploy-image \
    --repo-name=pso-infra-cicd-docker-scan-demo \
    --repo-owner=REPOSITORYOWNER \
    --branch-pattern=^main$ \
    --build-config=cloudbuild_deploy.yaml
```
* Create an artifact repository:
```bash
gcloud artifacts repositories create http-build-repo --repository-format=docker \
--location=us-central1 --description="Repository for scan and build"
```



## Usage

1. Clone your current repository i.e. ```git clone <repo url>```
1. Create a branch ```git checkout -b test-deploy```
1. Add the following Dockerfile:
```bash
cat << EOF > Dockerfile
FROM httpd:alpine

EOF
```
4. Add the change to your repository:
``` bash
git add --all
git commit -m"submitting a container for deployment"
git push origin test-deploy
```
5. From the github console create a [pull request](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request), make sure the base is your local main branch and not the forked one.
1. The build should fail
1. Add the patching
```
echo "RUN apk add --upgrade apr" >> Dockerfile
```
8. commit the changes to the branch 
``` bash
git add --all
git commit -m"Patching container"
git push origin test-deploy
```
9. Verify the build passes 
1. [Merge](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/incorporating-changes-from-a-pull-request/merging-a-pull-request) to the main branch the cloud build should run and push the image to artifact registry
