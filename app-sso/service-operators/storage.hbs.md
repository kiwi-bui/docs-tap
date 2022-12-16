# Storage

AppSSOs `AuthServer` handles data pertaining to user's session, identity and access tokens, and approved or rejected
consents. For production environments, it is critical to provide your own storage source to enable enterprise
functions such as data backup and recovery, auditing, and long-term persistence, as per your organization's data and
security policies.

AppSSO currently only supports Redis `v6.0` or above as a storage solution. Version 6.0 introduced TLS support to ensure
encrypted client-server communication -- AppSSO enforces TLS by default.

<p class="note">
<strong>Note:</strong>
[Storage provided by default](#storage-provided-by-default) refers to an `AuthServer` resource in which the field
`.spec.storage` is not set.
</p>

<p class="note">
<strong>Note:</strong>
Although data in motion is encrypted by using TLS, data at rest is not encrypted by default through `AuthServer`. Each
storage provider is responsible for encrypting their own data. See [data types](#data-types) for more
information about storage.
</p>

<p>
<strong>Best Practice for Securing Data at Rest:</strong>
To be compliant with HIPAA, FISMA, PCI and GDPR, you must encrypt data at rest. Securing
the underlying infrastructure that Redis uses is crucial to protect against a potential attack.
The National Institute for Standards and Technology – Federal Information Processing Standards (NIST-FIPS) sets the
standard for best practice when it comes to data security in the US.
Symmetric cryptography can be used to protect data at rest. This means that the same key encrypts and
decrypts the data, so there is no need for a different private and public key. The [Advanced Encryption Standard (AES)](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197.pdf)
encryption algorithm is an industry standard for securing data at rest. For the highest level security, VMware recommends
to use a 256-bit key.
</p>

## Configuring Redis

To configure Redis as authorization server storage, you must have the following details about your Redis server:

* **Server CA certificate** (optional) - the Certificate Authority (CA) certificate used to verify Redis TLS
  connections. If Redis Server certificate is signed by a public CA, providing this certificate is not required.
* **host** (required) - the domain name, IP address, or host name of your Redis server.
* **port** (optional) - the port number of your Redis server (default: `6379`). Must be a string.
* **username** (optional) - the username used to authenticate against your Redis server.
* **password** (optional) - the password used to authenticate against your Redis server.

AppSSO takes _secure-by-default_ approach and does not establish non-encrypted communication channels.
The `AuthServer` resource enters an error state if a non-encrypted connection is attempted.

_mTLS_ is not supported, however Vanilla Redis uses _mTLS_ by default. It can be turned off by setting `tls-auth-clients no`.
For more information, see [Redis documentation](https://redis.io/docs/management/security/encryption/#client-certificate-authentication).


The following steps introduce the path to configuring Redis with AppSSO:

1. [Configuring Redis Server CA certificate](#configuring-redis-server-ca-certificate)
1. [Configuring a Redis Secret](#configuring-a-redis-secret)
1. [Attaching storage to an `AuthServer`](#attaching-storage-to-an-authserver)

### Configuring Redis Server CA certificate

If your Redis includes a custom or non-public Server CA certificate, you must instruct AppSSO to
trust the CA certificate. This is required for the authorization server to communicate with your
Redis over TLS. See [CA certificates](ca-certs.hbs.md) for more information about configuring a CA certificate with AppSSO.

### <a id='configuring-a-redis-secret'></a>Configuring a Redis Secret

To provide _coordinates_ (the location details) of your Redis server, you have to create a `Secret` resource that
follows well-known Secret entries conventions specified
in [Service Bindings 1.0.0 specification](https://github.com/servicebinding/spec#well-known-secret-entries).

Here is an example of a properly formatted `Secret` resource:

<p class="note">
<strong>Note:</strong>
The Secret **must** be created in the same namespace as your `AuthServer`
</p>

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: redis-credentials
  namespace: my-authserver
type: servicebinding.io/redis        # required
stringData:
  type: "redis"                      # required, must equal 'redis'
  ssl: "true"                        # required, must equal 'true'
  host: "redis01.prod.example.com"   # required
  port: "6379"                       # optional, must be a string, defaults to "6379" if left empty
  password: "!!veryStrongPassword!!" # optional
  username: "redis01-user"           # optional
```

### Attaching storage to an AuthServer

Once a Redis Secret resource is applied, you may reference the Secret in `.spec.storage`. Here is an example of an
AuthServer with a reference to a Redis Secret:

```yaml
apiVersion: sso.apps.tanzu.vmware.com/v1alpha1
kind: AuthServer
metadata:
  name: my-authserver-example
  namespace: my-authserver
spec:
  # ...
  storage:
    redis:
      serviceRef:
        apiVersion: "v1"
        kind: "Secret"
        name: redis-credentials
```

After `AuthServer` is applied, ensure its `Status` is `Ready`.

### Inspecting storage of an AuthServer

You can inspect the status of an `AuthServer`'s storage as follows:

```bash
kubectl get authserver <authserver-name> \
  --namespace <authserver-namespace> \
  --output jsonpath="{.status.storage.redis}" | jq
```

Expect to see the following output with actual Redis host and port:

```json
{
  "redis": {
    "host": "ci-redis.authservers.svc.cluster.local",
    "port": "6379"
  }
}
```

## Storage provided by default

If no storage is defined, an `AuthServer` provides its own short-lived ephemeral storage solution,
Redis. The provided Redis is configured to never flush any data to any volume that may be attached to the Pods operating
the authorization server.

<p class="note caution">
<strong>Caution:</strong>
The default storage configuration is most useful in prototyping or testing environments, and **should not be relied on
in production environments.**
</p>

To view details for Redis of an `AuthServer`:

```shell
# Get the Redis image
kubectl get authserver <authserver-name> \
  --namespace <authserver-namespace> \
  --output jsonpath="{.status.deployments.redis}" | jq

# Get the Redis host and port
kubectl get authserver <authserver-name> \
  --namespace <authserver-namespace> \
  --output jsonpath="{.status.storage.redis}" | jq
```

## <a id='data-types'></a>Data types

The following data is stored in Redis:

- Client information
    - Authorization grant type
    - Client id

- User session
    - Session token
    - Refresh token

- Identity and access tokens

    >**Note** This is the data that carries the highest level risk.

    - Authentication token includeing the principal
        - Personally identifying information such as email and name

- Approved or rejected consents
    - A client identifier
    - A reference to the user
    - A list of the Authorities that the user has granted to this client

## Known limitations of storage providers 

### Redis Cluster

When your storage is provided by _Redis Cluster_, additional settings might be required.

The nodes and the maximum number of redirects must be set in your _Service Bindings_ `Secret`.
For example, in addition to the entries described by [how to configure a Redis `Secret`](#configuring-a-redis-secret),
you must provide `cluster` settings:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: redis-cluster-credentials
  namespace: authservers
type: servicebinding.io/redis
stringData:
  #...
  cluster.max-redirects: 5
  cluster.nodes: 100.90.1.10:6379,100.90.1.11:6379,100.90.1.12:6379
```

<p class="note">
<strong>Note:</strong>
`cluster.nodes` must be a comma-separated list of `<ip>:<port>`.
</p>
