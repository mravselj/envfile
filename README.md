# secrets-to-dotenv2

[![e2e](https://github.com/mravselj/envfile/actions/workflows/e2e.yml/badge.svg)](https://github.com/mravselj/envfile/actions/workflows/e2e.yml)

This is a fork of [secrets-to-env-action](https://github.com/thaind0/envfile)

This action provides the following functionality for GitHub Actions users:

- Read Github secrets and export **all** of them as environment variables and write them to a file
- Optionally including, excluding and manipulating variables as needed before importing
- Additionally enable option to remove secrets prefix before exporting to env and .env. Useful for DEVELOPMENT_*, STAGING_*, PRODUCTION_* prefixes

<table>
<tr>
<th>
Before
</th>
<th>
After
</th>
</tr>
<tr>
<td>
<pre>
- run: echo "Value of MY_SECRET1: $MY_SECRET1"
  env:
    MY_SECRET1: ${{ secrets.MY_SECRET1 }}
    MY_SECRET2: ${{ secrets.MY_SECRET2 }}
    MY_SECRET3: ${{ secrets.MY_SECRET3 }}
- run: |
    echo "MY_SECRET1=${{ secrets.MY_SECRET1 }}" >> .env
    echo "MY_SECRET2=${{ secrets.MY_SECRET2 }}" >> .env
    echo "MY_SECRET3=${{ secrets.MY_SECRET3 }}" >> .env
    ...
</pre>
</td>

<td>
<pre>
- uses: mravselj/envfile@v1
  with:
    secrets: ${{ toJSON(secrets) }}
- run: echo "Value of MY_SECRET1: $MY_SECRET1"
</pre>
</td>

</tr>
</table>

## Usage

Add the following action to your workflow:

```yaml
- uses: mravselj/envfile@v1
  with:
    secrets: ${{ toJSON(secrets) }}
```

After running this action, subsequent actions will be able to access the secrets as env variables.
Note the `secrets` key. It is **mandatory** so the action can read and export the secrets.

**Basic:**

```yaml
steps:
- uses: actions/checkout@v3
- uses: mravselj/envfile@v1
  with:
    secrets: ${{ toJSON(secrets) }}
- run: echo "Value of MY_SECRET: $MY_SECRET"
```

**Custom file:**

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: mravselj/envfile@v1
    with:
      secrets: ${{ toJSON(secrets) }}
      file: .prod.env
```

**No environment variables:**

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: mravselj/envfile@v1
    with:
      secrets: ${{ toJSON(secrets) }}
      no_env: true
```

**No file:**

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: mravselj/envfile@v1
    with:
      secrets: ${{ toJSON(secrets) }}
      file:
```

**Include or exclude secrets:**

Exclude defined secret(s) from list of secrets (comma separated, supports regex).

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: mravselj/envfile@v1
    with:
      secrets: ${{ toJSON(secrets) }}
      exclude: MY_SECRET, MY_OTHER_SECRETS*
# MY_SECRET is not exported
```

**Only** include secret(s) from list of secrets (comma separated, supports regex).

```yaml
steps:
- uses: actions/checkout@v3
- uses: mravselj/envfile@v1
  with:
    secrets: ${{ toJSON(secrets) }}
    include: MY_SECRET, MY_OTHER_SECRETS*
- run: echo "Value of MY_SECRET: $MY_SECRET"
```

To export secrets that start with a given string, you can use `include: PREFIX_*`.

NOTE: If specified secret does not exist, it is ignored.

**Add a prefix:**

Adds a prefix to all exported secrets.

```yaml
steps:
- uses: actions/checkout@v3
- uses: mravselj/envfile@v1
  with:
    secrets: ${{ toJSON(secrets) }}
    prefix: PREFIXED_
- run: echo "Value of PREFIXED_MY_SECRET: $PREFIXED_MY_SECRET"
```

**Override:**

Overrides already existing variables (default is true)

```yaml
env:
  MY_SECRET: DONT_OVERRIDE
steps:
- uses: actions/checkout@v3
- uses: mravselj/envfile@v1
  with:
    secrets: ${{ toJSON(secrets) }}
    override: false
- run: echo "Value of MY_SECRET: $MY_SECRET"
Value of MY_SECRET: DONT_OVERRIDE
```

**Convert:**

Converts all exported secrets according to a [template](https://github.com/blakeembrey/change-case#core).
Available: `lower, upper, camel, constant, pascal, snake`.

```yaml
steps:
- uses: actions/checkout@v3
- uses: mravselj/envfile@v1
  with:
    secrets: ${{ toJSON(secrets) }}
    convert: lower
- run: echo "Value of my_secret: $my_secret"
```

**Include or skip the prefix on conversion (default is true):**

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: mravselj/envfile@v1
    with:
      secrets: ${{ toJSON(secrets) }}
      prefix: PREFIX_
      convert: lower
      convert_prefix: false
  - run: env
# E.g. secret with MY_SECRET would become PREFIX_my_secret
```

**Remove the prefix on conversion (default is true):**

```yaml
steps:
  - uses: actions/checkout@v3
  - uses: mravselj/envfile@v1
    with:
      secrets: ${{ toJSON(secrets) }}
      include: PREFIX_*
      remove_prefix: PREFIX_
      convert_prefix: false
  - run: env
# E.g. secret with PREFIX_MY_SECRET would be exported as MY_SECRET
```

## How it works

This action uses the input in `secrets` to read all the secrets in the JSON format, and exporting all the variables one by one.

## License

The scripts and documentation in this project are released under the [MIT License](LICENSE)
