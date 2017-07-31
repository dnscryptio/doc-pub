# Key Management #

This document describes the overall architecture and the details of
how we manage keys used for the dnscrypt.io service.

## Outline ##

The idea is that a single *provider key pair* is created on, resides
on, and newer leaves, a *key holder*, (Step 1). The key holder uses
the provider key pair to generate pairs of *service key* and a
*service certificate*, per service instance, with limited duration,
(Step 2 and Step 3). The instance key and certificate is pushed to a
*key storage*, (Step 4), and the service instances access service keys
in the key storage, (Step 5). This allows the key holder to be
isolated, and separates provider key pair storage from service
key/certificate access control. In further details the process for
managing keys is as follows:

 1. Generate provider key pair.

        dnscrypt-wrapper --generate-provider-keypair ...
        # Input: None (Randomness from host)
        # Output: public.key
        # Output: secret.key

    These keys are the cryptographic identity of the service. It is
    **important** to keep `secret.key` secret, as it can be used to
    generate service keys, and impose as part of dnscrypt.io.

 2. Generate service key.

        dnscrypt-wrapper --generate-crypt-keypair
        # Input: public.key
        # Input: secret.key
        # Output: <SERIAL>.key

    This key is "half" of the identity used by a `dnscrypt-wrapper`
    service instance for encryption. It is **important** to keep it
    secret, as it can be used to impose as the dnscrypt service
    instance, or to decrypt communication with instance and
    clients. `<SERIAL>` is a unique, strictly increasing serial for a
    service key/service certificate pair.

 3. Generate service certificate.

        dnscrypt-wrapper --generate-cert-file ...
        # Input: <SERIAL>.key
        # Input: public.key
        # Input: secret.key
        # Input: Duration of validity
        # Output: <SERIAL>.cert

    This certificate is the other "half" of the identity used by a
    `dnscrypt-wrapper` service instance for encryption. The
    certificate, `<SERIAL>.cert`, is intended for publication.

 4. Push service key and service certificate to key storage.
 
 5. Fetch service key and service certificate to to instance.
 
 6. Start service.
 
        dnscrypt-wrapper ...
        # Input: <SERIAL>.key
        # Input: <SERIAL>.cert
        # Output: None (Service is started)


## Implementation ##

### Key holder ###

### Key storage ###

Service key and certificates are stored in an AWS s3 bucket, with a
folder for each instance, like the following:

    <BUCKET>/<INSTANCE>/<SERIAL>.key
    <BUCKET>/<INSTANCE>/<SERIAL>.cert

Key holder has an *AWS access key* granting full access to the
bucket. Each service instance has an AWS access key granting only read
access to the relevant instance folder, i.e. `<BUCKET>/<INSTANCE>/`.

### Miscellaneous ###

 * `<SERIAL>` is `YYYYMMddhhmmss`
 * `<INSTANCE>` is ???
