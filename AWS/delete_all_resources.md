1. using `aws-nuke`

* create `config.yml` file

```yaml
regions:
  - global
  - us-east-1
  - eu-central-1
  # Add other regions if you use them
account-blocklist:
  - "999999999999"
accounts:
  "085710281662": # the account who owns the resources
    presets: []
```

* create aliase for the account

  ```
  bash
  aws iam create-account-alias --account-alias <alias-name>
  ```
* `docker run --rm -it   -v $(pwd)/config.yml:/home/aws-nuke/config.yml -e AWS_ACCESS_KEY_ID `<access_key>`   -e AWS_SECRET_ACCESS_KEY=`<secret_key>` rebuy/aws-nuke   --config /home/aws-nuke/config.yml --no-dry-run`
