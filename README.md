Heroku buildpack: Node.js
=========================

This is a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpack) for Node.js apps.
It uses [NPM](http://npmjs.org/) and [SCons](http://www.scons.org/).

Usage
-----

Example usage:

    $ ls
    Procfile  package.json  web.js

    $ heroku create --stack cedar --buildpack http://github.com/heroku/heroku-buildpack-nodejs.git

    $ git push heroku master
    ...
    -----> Heroku receiving push
    -----> Fetching custom buildpack
    -----> Node.js app detected
    -----> Vendoring node 0.4.7
    -----> Installing dependencies with npm 1.0.8
           express@2.1.0 ./node_modules/express
           ├── mime@1.2.2
           ├── qs@0.3.1
           └── connect@1.6.2
           Dependencies installed

The buildpack will detect your app as Node.js if it has the file `package.json` in the root.  It will use NPM to install your dependencies, and vendors a version of the Node.js runtime into your slug.  The `node_modules` directory will be cached between builds to allow for faster NPM install time.

Hacking
-------

To use this buildpack, fork it on Github.  Push up changes to your fork, then create a test app with `--buildpack <your-github-url>` and push to it.

To change the vendored binaries for Node.js, NPM, and SCons, use the helper scripts in the `support/` subdirectory.  You'll need an S3-enabled AWS account and a bucket to store your binaries in.

For example, you can change the vendored version of Node.js to v0.5.8.

Before following the steps below, you'll need a Heroku build server to manage building and packaging Node for use by your custom buildpack. First, install the 'vulcan' gem

	$ gem install vulcan
	
Then, from your custom buildpack directory, create a build server:

	$ vulcan build yourAppName

Once that process is complete, you'll need to build a Heroku-compatible version of Node.js:

    $ export AWS_ID=xxx AWS_SECRET=yyy S3_BUCKET=zzz
    $ s3 create $S3_BUCKET
    $ support/package_node 0.5.8

Then, create a package for npm:

	$ support/package_npm 1.0.94

Open `bin/compile` in your editor, and change the following lines (make sure that the npm and Node versions are compatible):

    NODE_VERSION="0.5.8"
	NPM_VERSION="1.0.94"

    S3_BUCKET=zzz

Commit and push the changes to your buildpack to your Github fork, then push your sample app to Heroku to test.  You should see:

    -----> Vendoring node 0.5.8
