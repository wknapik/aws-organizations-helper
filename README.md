# AWS organizations helper

This script aims to make working with multiple AWS subaccounts more convenient.

`aws-organizations-helper` reads stdin in one of multiple formats and produces
output on stdout in one of multiple formats.

Input can either be `aws-config` (the ini format of ~/.aws/config), or `aws-api`
(the json output of `aws organizations list-accounts --output json`).

Output can be `aws-config` (a set of profiles that can be appended to
~/.aws/config), `cookie-raw` (a cookie with a list of accounts, that can be
injected into a browser session to populate the role history dropdown), or
`links-raw` (a list of links to AWS console to assume a role in each account).

## Requirements

bash, coreutils, util-linux,
* awscli (only `--output-format cookie-raw`)
* jq (only `--input-format aws-api`)

## Help
```
USAGE: aws-organizations-helper [options]
Process information about AWS organizations
  -i | --input-format ifmt      input format [aws-api]
  -o | --output-format ofmt     output format [aws-config]
  -r | --default-role role      default role [OrganizationAccountAccessRole]
  -h | --help                   print this help and exit

ifmt = aws-config | aws-api
ofmt = aws-config | cookie-raw | links-raw

E.g.:
% aws organizations list-accounts|aws-organizations-helper >>~/.aws/config
% aws-organizations-helper -i aws-config -o links-raw <~/.aws/config
```

## Read AWS API

### Print URLs

```
% aws organizations list-accounts|aws-organizations-helper -i aws-api -o links-raw
https://signin.aws.amazon.com/switchrole?account=1337&roleName=OrganizationAccountAccessRole&displayName=some_account&color=F2B0A9
[...]
%
```

### Print a cookie

```
% aws organizations list-accounts|aws-organizations-helper -i aws-api -o cookie-raw
WARNING: AWS console will only display the first 5 profiles.
noflush_awsc-roleInfo=%7B%22bn%22%3A%22%22%2C%22ba%22%3A%221337%22%2C%22rl%22%3A%5B%7B%22a%22%3A%221338%22%2C%22r%22%3A%22OrganizationAccountAccessRole%22%2C%22d%22%3A%22some_account%22%2C%22c%22%3A%229A8A40%22%7D%5D%7D%0A
%
```

### Print a config

```
% aws organizations list-accounts|aws-organizations-helper -i aws-api -o aws-config
[profile some_account]                                                                                                                                                                                                                      
role_arn = arn:aws:iam::1337:role/OrganizationAccountAccessRole                                                                                                                                                                        
source_profile = default
[...]
%
```

## Read AWS config

### Print URLs

```
% aws-organizations-helper -i aws-config -o links-raw <~/.aws/config
https://signin.aws.amazon.com/switchrole?account=1337&roleName=OrganizationAccountAccessRole&displayName=some_account&color=F2B0A9
[...]
%
```

### Print a cookie

```
% aws-organizations-helper -i aws-config -o cookie-raw <~/.aws/config
WARNING: AWS console will only display the first 5 profiles.
noflush_awsc-roleInfo=%7B%22bn%22%3A%22%22%2C%22ba%22%3A%221337%22%2C%22rl%22%3A%5B%7B%22a%22%3A%221338%22%2C%22r%22%3A%22OrganizationAccountAccessRole%22%2C%22d%22%3A%22some_account%22%2C%22c%22%3A%229A8A40%22%7D%5D%7D%0A
%
```

### Print a config

```
# Doesn't make much sense to run, other than to verify that the helper works correctly.
% aws-organizations-helper -i aws-config -o aws-config <~/.aws/config
[profile some_account]                                                                                                                                                                                                                      
role_arn = arn:aws:iam::1337:role/OrganizationAccountAccessRole                                                                                                                                                                        
source_profile = default
[...]
%
```
