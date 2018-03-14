<a href="https://www.sparkpost.com"><img src="https://www.sparkpost.com/sites/default/files/attachments/SparkPost_Logo_2-Color_Gray-Orange_RGB.svg" width="200px"/></a>

[Sign up](https://app.eu.sparkpost.com/sign-up?src=Dev-Website&sfdcid=70160000000pqBb) for a SparkPost account and visit our [Developer Hub](https://developers.sparkpost.com) for even more content.

<div class="alert alert-info">SparkPost EU is now available in the EU region. You can now <a href="https://app.sparkpost.com/sign-up?src=Dev-Website&sfdcid=70160000000pqBb">Sign up</a> for a SparkPost EU account.</div>

# SparkPost API Documentation

[![Run in Postman](https://s3.amazonaws.com/postman-static/run-button.png)](https://www.getpostman.com/run-collection/81ee1dd2790d7952b76a)

## Prerequisites

* You have installed [Git](http://git-scm.com/downloads) on your development machine
* You have installed [Node.js and NPM](https://nodejs.org/) on your development machine

The SparkPost API Docs in API Blueprint format (based on Markdown)

## Installation

1. Clone the repository

```git clone https://github.com/SparkPost/sparkpost-api-documentation```

2. Change into the directory

2. We use [Grunt](http://gruntjs.com/) for our task runner, so you will also have to install Grunt globally

```npm install -g grunt-cli```

3. Install the dependencies

```npm install```

## Development

### Grunt Commands

#### API Blueprint Validator

Once all the dependencies are installed, you can execute the API Blueprint Validator tests in the following ways:

* Run the test on ALL /services/ files sequentially
  ```grunt test```

* Run the test an individual /services/ file
  ```grunt shell:test:<filename>```

#### Static Docs Development Workflow

You can use `grunt preview` to generate API docs under `static/` and start an auto-regen watch process.

*Note: the output of this command is not identical to production renders. Its intended as a usable preview for development work.*

### Deploying

The API documentation lives at https://developers.sparkpost.com/api/. When a commit is made to the `master` branch of this repo, it triggers a Travis CI run that executes the code in `bin/copy-to-devhub.sh`. The automated workflow is as follows:

* Clone the develop branch of the [DevHub](https://github.com/SparkPost/developers.sparkpost.com) repo
* Generate the HTML files for the API docs into `developers.sparkpost.com/_api`
* Commit the changes, which triggers a Travis CI build for the `developers.sparkpost.com` repo

The Travis CI projects can be viewed here:

* [sparkpost-api-documentation](https://travis-ci.org/SparkPost/sparkpost-api-documentation)
* [developers.sparkpost.com](https://travis-ci.org/SparkPost/developers.sparkpost.com)

### Contributing
[Guidelines for adding issues](CONTRIBUTING.md#guidelines-for-adding-issues-to-sparkpost-api-documentation)

[API Blueprint Specification](https://github.com/apiaryio/api-blueprint/blob/master/API%20Blueprint%20Specification.md)

[Submitting pull requests](CONTRIBUTING.md#contribution-steps)

## License

Read the [complete license](/LICENSE) information for our Sparkpost API documentation.
