# WIP - Hashicorp VAULT

* Secrets management
  - version control
  - rotate

* Data protection
  - Encrytion keys
  - Certificates

* clients <-> API <-> VAULT (vault authenticates the client, and if it's valid it returns a token which can be used )

  each client require a policy in VAULT, which determine what a client is able to do 


Comparison:
* Azure Key Vault
* AWS Key management service


# Installation

Linux: wget https://relases.hashicorp.com/vault/{version}/vault_{version}.zip


- Secret Engines, secret is installed by default (it's key, value store)
vault kv put storepath key=value


--> Secret lifecycle
  Create
  Read
  Update
  Delete
    - Soft, deleted by recoverable
    - Hard
  Metadata destroy

  ### Key Value Secrets Engine

  Version 1:
    - No versionning
    - Faster
    - Deleted items are gone (no recoverable)
    - Can be upgrated to version 2
    - This is the default version on creation of new Engine
  Version 2:
    - it has versioning included, by default stores 10 version of it.
    - Possibly less performant
    - Delete items and metadata retained
    - Can not be downgrade

  
  in order to remove a secret for good. You must delete the metadata of the secret as well.


Secret Engines:
 responsible for: Store, generate and encrypt data

 types: AWS, Database, Active Directory, PKI, SSH

 lifecycle--->  Enable > Disable > Move > Tune

 -- Enable or Disable secret engine:

 ```
vault secrets enable [options] TYPE
 ```

### Vault Paths

API rules everything and all are based on paths

* path-help, for the CLI

## Dynamic Secrets

Secrets created on demand

-> *lease* issue for each secret and validity period
-> Control renewal and revocation


## Authentication Methods

- internal Authentication: user and pass
- external Authentication: Ldap, Okta, AWS access management, Azure access, etc
  
  Similar process:
    * Enable auth method
    * Write configuration
    * Add a role to the auth method
    * Disable auth method

### Testing userpass auth

```
vault auth enable userpass
```

checking all api endpoints (**path-help**)


```
vault path-help auth/userpass
```

## Vault policies

#### Policies define **WHO, WHAT and HOW** access/manage resources

You can use HCL or JSON to define the policies.

Policies are apply to *paths*

### Policy Document

```hcl
path "path_of_secret_data/[*]" {
    capabilities = ["create", "read", "update"]

    required parameters = ["param_name"]

    allowed parameters = {
        param_name = ["list", "of", "values"]
    }

    denied parameters = {
        param_name = ["list", "of", "values"]
    }
}
```

```
vault policy list
```

... update policy, 

via path:
```
vault write sys/policy/[policy] policy=[policy_file.hcl]
```

- entity_id?

## Managing client tokens

### Tokens

Include Policies and Metadata

#### Types
- Service Token
    * More flexible token, 
- Batch Token
    * ligthweight 

- There is also Token hierarchy, if 1 token at the top is removed, all children tokens are orphan
- Token accessors, reads the metadata of the token but not the token itself.


#### Response wrapping

Combine responses and save them in a *cubbyhole* in order to generate a new Token which will grant you access to the resources. The client uses the 'new token' to talk with the *cubbyhole* and then from there, retrieves the information necessary.

How to create it?

* generate a 'normal' secret
* retrieve the secret with *wrap-ttl* option

```bash
vault kv put secret/app-server api-key=5555
```

```bash
# 300 in seconds
vault get -wrap-ttl=300 secret/app-server
```

then you will get a *wrapped Token*, the next step is do a POST request with this token in order to retrieve the 'original' information

POST -> VAULT_ADDR/v1/sys/wrapping/unwrap

## Installation

TODO


## Auditing

log types request and responses, available options are: json, syslog, socket

```bash
vault audit enable -path=vault_path [type]
```

