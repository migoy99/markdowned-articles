In this tutorial, we'll walk through a GitHub Actions workflow designed to analyze the code of a Laravel application using PHPStan (a static analysis tool) and run PHPUnit tests. This workflow helps catch potential issues early in development.

## Trigger on Code Changes

The workflow is set to trigger whenever changes are pushed to the "develop" branch of your repository.

```yaml
on:
  push:
    branches:
      - develop
```

This ensures that our code is analyzed and tests are run whenever there's a new commit to the development branch.

## Job Configuration

The workflow defines a single job called "ptpstan-phpunit." This job runs on the latest Ubuntu environment and includes a MySQL service container for database-related tasks during testing.

```yaml
jobs:
  ptpstan-phpunit:
    runs-on: ubuntu-latest
    services:
      mysql:
        image: mysql:8.0
        # ...
```

The MySQL service container is essential for running PHPUnit tests that interact with a database.


## Workflow Steps

- **Checkout Code**: This step fetches the latest code from your GitHub repository.

```yaml
- name: Checkout Code
  uses: actions/checkout@v3
```

- **Copy .env.testing to .env**: This copies the testing environment configuration to ensure proper settings during testing.

```yaml
- name: Copy .env.testing to .env
  run: php -r "file_exists('.env') || copy('.env.testing', '.env');"
```

- **Validate Composer Files**: This step ensures that the Composer configuration files are correct.

```yaml
- name: Validate composer.json and composer.lock
  run: composer validate --strict
```

- **Cache Composer Packages**: Caching Composer packages speeds up workflow execution by reusing previous installations.

```yaml
- name: Cache Composer packages
  id: composer-cache
  uses: actions/cache@v3
  with:
    path: vendor
    key: ${{ runner.os }}-php-${{ hashFiles('**/composer.lock') }}
    restore-keys: |
      ${{ runner.os }}-php-
```

- **Install Composer Dependencies**: This installs the necessary PHP dependencies for your Laravel application.

```yaml
- name: Install composer dependencies
  run: composer install --prefer-dist --no-progress
```

- **Run PHPStan Static Code Analysis**

```yaml
- name: Run PHPStan Static Code Analysis
  run: ./vendor/bin/phpstan analyse app config database public routes/web.php tests resources --memory-limit=2g
```

- `./vendor/bin/phpstan`: This references the PHPStan executable located in the `vendor` directory, which is where Composer installs dependencies.
    
- `analyse`: This is the command used to perform static code analysis with PHPStan.
    
- `app config database public routes/web.php tests resources`: These are the directories and files you want PHPStan to analyze. You can customize this list based on your project structure.

- PHPStan recommends analyzing only files with the code you've written yourself. This is why we exclude the `vendor` directory where third-party dependencies are stored. You usually can't fix issues in external libraries, and analyzing them may lead to noise in the results.

- `--memory-limit=2g`: This option sets the memory limit for PHPStan. The `2g` stands for 2 gigabytes. Adjust this value based on the available memory in your environment.

- You can also add PHPStan levels to control the strictness of analysis. For example: `--level 1`: This sets PHPStan to level 1, which is a lower strictness level. You can use levels 0 to 8, with 8 being the strictest.

- **Generate Key**: This command generates a unique key for your Laravel application.

```yaml
- name: Generate key
  run: php artisan key:generate
```

- **Migrate Tables to MySQL Database**: This step migrates your database table.

```yaml
- name: Migrate Tables to MySQL Database
  env:
    DB_CONNECTION: mysql
    DB_DATABASE: test-db
  run: php artisan migrate
```

- **Run PHPUnit Tests**: PHPUnit executes your test suite, verifying that your application behaves as expected.

```yaml
- name: Run php unit test
  env:
    DB_CONNECTION: mysql
    DB_DATABASE: test-db
  run: php artisan test
```


## The entire PHPStan and PHPUnit workflow file

```yaml
# PHPStan and PHPUnit Workflow

name: phpstan-analyze-phpunit-test

on:
  push:
    branches:
      - develop

jobs:
  ptpstan-phpunit:
    runs-on: ubuntu-latest
    services:
      mysql:
        image: mysql:8.0
        env:
          MYSQL_DATABASE: test-db
          MYSQL_ROOT_PASSWORD: pass
        ports:
          - 3306:3306
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=3
    
    steps:
    - name: Checkout Code
      uses: actions/checkout@v3

    - name: Copy .env.testing to .env
      run: php -r "file_exists('.env') || copy('.env.testing', '.env');"

    - name: Validate composer.json and composer.lock
      run: composer validate --strict

    - name: Cache Composer packages
      id: composer-cache
      uses: actions/cache@v3
      with:
        path: vendor
        key: ${{ runner.os }}-php-${{ hashFiles('**/composer.lock') }}
        restore-keys: |
          ${{ runner.os }}-php-

    - name: Install composer dependencies
      run: composer install --prefer-dist --no-progress

    - name: Run PHPStan Static Code Analysis
      run: ./vendor/bin/phpstan analyse app config database public routes/web.php tests resources --memory-limit=2g

    - name: Generate key
      run: php artisan key:generate

    - name: Migrate Tables to MySQL Database
      env:
        DB_CONNECTION: mysql
        DB_DATABASE: test-db
      run: php artisan migrate

    - name: Run php unit test
      env:
        DB_CONNECTION: mysql
        DB_DATABASE: test-db
      run: php artisan test
```

Copy and paste this YAML code into your GitHub Actions workflow file. This workflow performs static code analysis with PHPStan and runs PHPUnit tests whenever changes are pushed to the "develop" branch of your Laravel application. Feel free to customize it based on your specific project needs!

## Optionally, Run Separate PHPStan Analysis Job

Optionally, if you prefer to have a separate job for PHPStan analysis, you can add the following job to your workflow file:

```yaml
phpstan-analyze:
  runs-on: ubuntu-latest
  steps:
  - name: Checkout Code
    uses: actions/checkout@v3
  - name: Install Composer Dependencies
    uses: php-actions/composer@v6
    with:
      php_version: "8.2"
      version: "2.5.8"
  - name: Run PHPStan Static Code Analysis
    uses: php-actions/phpstan@v3
    with:
      path: app/ config/ database/ public/ routes/web.php tests/ resources/
      memory-limit: 2g
```

This job specifically runs PHPStan static code analysis using the official PHPStan GitHub Action. You can customize the `path` parameter to include the directories you want to analyze.

---

In this tutorial, we've walked through setting up a GitHub Actions workflow for a Laravel application. The workflow automatically checks out your code, validates Composer files, caches dependencies, and installs them. Optionally, you can include PHPStan static code analysis within the main job, or if you prefer, add a separate job dedicated to PHPStan. Feel free to tailor this workflow to suit your specific project needs.