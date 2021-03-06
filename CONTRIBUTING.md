All work on SimpleLogin happens directly on GitHub.

### Run code locally

The project uses
- Python 3.7+ and [poetry](https://python-poetry.org/) to manage dependencies
- Node v10 for front-end.

First, install all dependencies by running the following command.
Feel free to use `virtualenv` or similar tools to isolate development environment.

```bash
poetry install
```

On Mac, sometimes you might need to install some other packages like

```bash
brew install pkg-config libffi openssl postgresql
```

You also need to install `gpg`, on Mac it can be done with:

```bash
brew install gnupg
```

Then make sure all tests pass

```bash
pytest
```

Install npm packages

```bash
cd static && npm install
```

To run the code locally, please create a local setting file based on `example.env`:

```
cp example.env .env
```

Make sure to uncomment the `RESET_DB=true` to create the database locally.

Feel free to custom your `.env` file, it would be your default setting when developing locally. This file is ignored by git.

You don't need all the parameters, for example, if you don't update images to s3, then
`BUCKET`, `AWS_ACCESS_KEY_ID` can be empty or if you don't use login with Github locally, `GITHUB_CLIENT_ID` doesn't have to be filled. The `example.env` file contains minimal requirement so that if you run:

```
python3 server.py
```

then open http://localhost:7777, you should be able to login with the following account

```
john@wick.com / password
```

### Database migration

The database migration is handled by `alembic`

Whenever the model changes, a new migration has to be created.

If you have Docker installed, you can create the migration by the following script:

```bash
sh new_migration.sh
```

Make sure to review the migration script before committing it.
Sometimes (very rarely though), the automatically generated script can be incorrect.

We cannot use the local database to generate migration script as the local database doesn't use migration.
It is created via `db.create_all()` (cf `fake_data()` method). This is convenient for development and
unit tests as we don't have to wait for the migration.

### Code structure

The repo consists of the three following entry points:

- wsgi.py and server.py: the webapp.
- email_handler.py: the email handler.
- cron.py: the cronjob.

Here are the small sum-ups of the directory structures and their roles:

- app/: main Flask app. It is structured into different packages representing different features like oauth,  api, dashboard, etc.
- local_data/: contains files to facilitate the local development. They are replaced during the deployment.
- migrations/: generated by flask-migrate. Edit these files will be only edited when you spot (very rare) errors on the database migration files.
- static/: files available at `/static` url.
- templates/: contains both html and email templates.
- tests/: tests. We don't really distinguish unit, functional or integration test. A test is simply here to make sure a feature works correctly.

The code is formatted using https://github.com/psf/black, to format the code, simply run

```
black .
```

### Test sending email

[swaks](http://www.jetmore.org/john/code/swaks/) is used for sending test emails to the `email_handler`.

[mailcatcher](https://github.com/sj26/mailcatcher) is used to receive forwarded emails.

There are several steps to set up the email handler

1) run mailcatcher

```bash
mailcatcher
```

2) Make sure to set the following variables in the `.env` file

```
NOT_SEND_EMAIL=true
POSTFIX_SERVER=localhost
POSTFIX_PORT=1025
```

3) Run email_handler

```bash
python email_handler.py
```

4) Send a test email

```bash
swaks --to e1@d1.localhost --from hey@google.com --server 127.0.0.1:20381
```

Now open http://localhost:1080/, you should see the test email.