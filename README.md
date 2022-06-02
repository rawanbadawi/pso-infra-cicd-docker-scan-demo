# Container Image Build with scanning quickstart

This repository shows an example of building a container image while leveraging Google Cloud's artifact scanning for docker images.

## Pre-requisites
* A Google Cloud Project
* gcloud cli installed
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

* Install the google cloudbuild github app: https://cloud.google.com/build/docs/automating-builds/build-repos-from-github#installing_gcb_app

* create a cloudbuild trigger from the repository:
```
gcloud beta builds triggers create github \
    --name=test-docker \
    --repo-name=pso-infra-cicd-docker-scan-demo \
    --repo-owner=REPOSITORYOWNER \
    --pull-request-pattern=^main$ \
    --build-config=cloudbuild_test.yaml \
    --comment-control=COMMENTS_ENABLED_FOR_EXTERNAL_CONTRIBUTORS_ONLY
```
```
gcloud beta builds triggers create github \
    --name=deploy-docker \
    --repo-name=pso-infra-cicd-docker-scan-demo \
    --repo-owner=REPOSITORYOWNER \
    --branch-pattern=^main$ \
    --build-config=cloudbuild_deploy.yaml \
    --comment-control=COMMENTS_ENABLED_FOR_EXTERNAL_CONTRIBUTORS_ONLY
```
* Create an artifact repository:
```bash
gcloud artifacts repositories create http-build-repo --repository-format=docker \
--location=us-central1 --description="Repository for scan and build"
```



## Usage

1. In github create a branch and a pull request
1. Verify the build fails due to a vulnerability
1. Patch the vulnerability in apr by uncommenting the following line in the Dockerfile  
`# RUN apk add --upgrade apr`
1. commit the changes to the branch
1. Verify the build passes 
1. Merge to main the cloud build should run and push a registry
