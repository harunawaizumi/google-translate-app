# Cloud Translation API app

## Description

This is my experiment to demonstrate how to call Google APIs from Google Compute Engine and Google Cloud serverless compute platforms.
Deploying the same app to Compute Engine, App Engine, Cloud Functions, Cloud Run and locally is possible without code changes (all done in configuration)
Followed by this tutorial [Deploy a basic "Google Translate" on App Engine, Cloud Functions and Cloud Run](https://codelabs.developers.google.com/codelabs/cloud-nebulous-serverless-nodejs#0)

## Hosting options

- **Local** -- use localhost to run the app.
- **[Compute Engine](https://cloud.google.com/products/compute?hl=en)** -- create and run online VMs on high-performance, reliable cloud infrastructure. (app-hosting in the cloud: "IaaS") This would be your option when you need a server control or VMs level full control.
- **[App Engine](https://cloud.google.com/appengine)** (standard environment) — fully-managed serverless service. Develoeprs don't need to maintain infrastructure. You can use Node.js, Java, Ruby, C#, Go, Python or PHP. (app-hosting in the cloud; "PaaS")
  - _App Engine_ is for users who wish to deploy a traditional (but not containerized) web application direct from source code.
- **[Cloud Functions](https://cloud.google.com/functions)** — cloud-hosted functions or microservices ("FaaS"). It's suitable for stateless, short-live, event driven services.
  - If your app is relatively simple, is a single function, or perhaps some event-driven microservice, _Cloud Functions_ may be the right platform for you.
- **[Cloud Run](https://cloud.run)** — fully-managed serverless container-hosting in the cloud ("CaaS") service
  - If your apps are containerized or you have containerization as part of your software development workflow, use _Cloud Run_.

## Deployments and their files

| File                                           | Description                                                     |
| ---------------------------------------------- | --------------------------------------------------------------- |
| [`index.js`](index.js)                         | main application file                                           |
| [`templates/index.html`](templates/index.html) | application HTML template                                       |
| [`package.json`](package.json)                 | 3rd-party package requirements file                             |
| [`app.yaml`](app.yaml)                         | App Engine configuration file (only for App Engine deployments) |
| [`test/test_neb.js`](test/test_neb.js)         | unit tests (`mocha` &amp; `supertest`)                          |
| [`.gcloudignore`](.gcloudignore)               | files to exclude deploying to the cloud (administrative)        |
| `README.md`                                    | this file (administrative)                                      |

## Local Hosting

- Create service account: service account is an identity that an instance or an application can use to run API requests on your behalf.It's authenticated using a private-public key pair.
- Create a public/private key pair for the service account
- Assign roles to the service account

```
$ gcloud projects add-iam-policy-binding PROJECT_ID  --member=user:SERVICE_ACCOUNT --role=roles/cloudtranslate.user
```

- Download the service account key to a local machine: the public portion is stored on Google Cloud, while the private portion is available only to you.
- Set the environment vaiable `GOOGLE_APPLICATION_CREDENTIALS` to the path of the JSON file that contains your credentials.
  For example:

```
$ export GOOGLE_APPLICATION_CREDENTIALS="/home/user/Downloads/service-account-file.json"
```

- Enable billing
- Enable API: Google API you want to use must be enabled.

**TL;DR:** application files (`index.js` &amp; `package.json`). Instructions:

1. **Run** `npm install` (to install packages locally)
1. **Run** `gcloud auth application-default login` to set your credentials
1. **Run** `npm start` to run locally

## **App Engine (Node 16)**

- Update app.yaml to use the latest version of Node.js

```
runtime: nodejs20
```

- Update timeout: it gets timedout if you deploy without updating the config

```
$ gcloud config set app/cloud_build_timeout 1200
```

- Update IAM and its Role: make sure target service account you are using has permission for App Engine. (It took time to figure it out.)
  This command shows which service account has which role.

```
$ gcloud projects get-iam-policy PROJECT_ID
```

- Deploy the app to App Engine

```
$ gcloud app deploy
```

## **Cloud Functions (Node 16)**

**TL;DR:** Uses only the application files. Instruction(s):

- Deploy the app to Cloud Functions
  You'll be prompted for the REGION if deploying a Cloud Function the first time.
  Cloud Functions can be deployed to different regions within a project, but once the region has been set for a function, it cannot be changed.

```
$ gcloud functions deploy translate --runtime nodejs16 --entry-point app --trigger-http --allow-unauthenticated
```

The command creates &amp; deploys a new HTTP-triggered Cloud Function named `translate`.
Cloud Functions is directed to call the application object, `app`, via `--entry-point`. During execution `translate()` is called by `app`.

## **Cloud Run (Node 16 via Cloud Buildpacks)**

**TL;DR:** Uses only the application files. Instruction(s):

1. **Run** `gcloud run deploy translate --allow-unauthenticated --platform managed --source .` to deploy to Cloud Run
   - You'll be prompted to provide a REGION unless you also add `--region REGION` on the cmd-line which will give you a full non-interactive deploy
   - Buildpack creates container images to run on Google Cloud Services. You don't need any docker or container knowledge.

## References

These are relevant links only to the app in this folder (for all others, see the [README one level up](../README.md):

- [Node.js App Engine quickstart](https://cloud.google.com/appengine/docs/standard/nodejs/quickstart)
- [Nodejs App Engine (standard environment) runtime](https://cloud.google.com/appengine/docs/standard/nodejs/runtime)
- [Node.js Cloud Functions quickstart](https://cloud.google.com/functions/docs/quickstart-nodejs)
- [Node.js Cloud Run quickstart](https://cloud.google.com/run/docs/quickstarts/build-and-deploy/nodejs)
- [Express.js](https://expressjs.com)

### Expected output

```
$ npm test

> cloud-nebulous-serverless-nodejs@0.0.1 test
> mocha test/test_neb.js

Listening on port 8080


  Our application
    ✔ GET / should result in HTML w/"translate" in the body
    ✔ POST / should have translated "hello world" correctly (140ms)
DONE


  2 passing (170ms)
```

### Useful Commands

List user managed service accounts (name, email, status)

```
$ gcloud iam service-accounts list

```

List users managed and Google-managed service accounts (name, email, status, roles)

```
$ gcloud projects get-iam-policy PROJECT_ID
```
