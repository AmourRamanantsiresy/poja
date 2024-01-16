# poja

**Your Java Infrawork. What is infrawork? It's like framework, but extended to the infrastructure running it!**

The Java framework is Spring Boot. The infrastructure on which it runs is AWS.
In the same way Spring Boot enforces a specific style for writing Java applications,
POJA enforces a specific style for running Spring Boot on AWS.

In particular, the whole stack is fully serverless: no activity then no payment.
Spring Boot itself is run on Lambda functions.
The persistence is backed by Aurora Postgres v1 or SQLite (on EFS).
An asynchronous stack is also embedded using EventBrige and SQS.

Additionally, POJA generates Github Actions for CI/CD, for health check, for formatting, for releases and for periodic database snapshoting.
More generally, POJA provides a way to operate the Spring Boot framework on AWS on a large scale, at the highest level of quality,
and yet very simply through a [CLI](https://github.com/hei-school/poja-cli).

POJA stands for POstgres JAva as, in the beginning, it only supported the generation of a Java Spring Boot connected to a POstgres.

## Usage

Create a new POJA stack in minutes using the [POJA CLI](https://github.com/hei-school/poja-cli).
Update an existing POJA stack in seconds using the same CLI!

Say good bye to the days where you had hard times launching new Java/Github/AWS stacks,
and even harder times trying to apply infrastructure fixes or infrastructure features you developped on a stack that you want to apply on other stacks!

## Features

### Git Workflow

poja is to be used à-la Gitflow but with only two branches/environments: prod and preprod.
The moment you push in one of these branches, CI/CD will be triggered.

### Code Quality

* A formatting script is embedded with POJA. Just run `./format.sh` and your whole code will be formatted using Google Java Format. To quote the GJF team: "There is no configurability as to the formatter's algorithm for formatting. This is a deliberate design decision to unify our code formatting on a single format."

* CI will fail whenever code coverage is below 80% (by default, but it's configurable). POJA code are not taken into account in the coverage computation, as they are marked by the annotation `@PojaGenerated`. More generally, any class annotated with `@...Generated` will be excluded from coverage computation.

### Asynchronous Tasks

POJA comes with a fully serverless asynchronous stack based on AWS Eventbrige events and on AWS SQS.
Creating a new asynchronous treatment is as simple as generating the corresponding event in `endpoint.event` using the code generation capabilities of Eventbrige,
say [UserCreated](https://github.com/hei-school/poja-base/blob/prod/src/main/java/com/company/base/endpoint/event/gen/UuidCreated.java),
then creating the service that will consume it by adding the `Service` suffix to the event name in `service.event`,
that is [UserCreatedService](https://github.com/hei-school/poja-base/blob/prod/src/main/java/com/company/base/service/event/UuidCreatedService.java) for the given example.

The link between the event and its consumer service is automatically done by POJA through means of reflection.
Note that unlike the synchronous stack, the asynchronous stack does _not_ benefit from SnapStart.
Hence it requires to launch a whole Spring Boot on each cold start.
We consider this acceptable as the treatment is asynchronous anyway.

### Persistence

#### Database

You can either choose a PostgreSQL running on Aurora v1, or an SQLite running on EFS. If you already have an existing database, and want POJA to use that, no problem.

#### Bucket

POJA comes with an S3 bucket and the associated Java class for handling it. We already coded [everything](https://github.com/hei-school/poja/blob/92eec460ed309349b4dcaab75fd30855ac38d7b0/src/main/java/school/hei/poja/file/BucketComponent.java#L24) for uploading, downloading and generating presigned URLs for sharing.

### Mailing

We have a [Mailer](https://github.com/hei-school/poja/blob/prod/src/main/java/school/hei/poja/mail/Mailer.java). One method `Mailer::send` and voilà, your email is sent.

### Specification and Client Generation

POJA can generate Typescript and Java clients from a given OpenAPI specification.

If you directly started implementing without specifying, first you shouldn't have done that, second POJA fortunately allows you to generate an OpenAPI specification from your implementation using Spring Doc.
