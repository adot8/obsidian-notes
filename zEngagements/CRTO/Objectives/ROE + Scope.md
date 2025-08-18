
Your objective is to simulate an advanced threat and gain access to a secure storage server - enc-fs-1.contoso.enclave. You must prove access by writing a text file to its hard disk at the location C:\rto.txt. The content does not matter.

> enc-fs-1.contoso.enclave.

## Authorised Target Spaces

- dublin.contoso.com
    - 10.10.123.0/23
- contoso.com
    - 10.10.120.0/24
- contoso.enclave
    - 10.10.124.0/24

## Restricted Hosts

- 10.10.x.254
- enc-dc-1.contoso.enclave
    - 10.10.124.1

## Explicit Restrictions

TTPs may be intrusive but not destructive or disruptive. The following activities are prohibited:

- Changing account passwords.
- Disabling host-based defences, including anti-virus and firewalls.

# Foothold

You can log into both the [Attacker Desktop](https://labclient.labondemand.com/Instructions/8d0a05c4-506f-4c15-bd58-f0231f1c0c11?showWhenStarting=1#) and [Foothold Machine](https://labclient.labondemand.com/Instructions/8d0a05c4-506f-4c15-bd58-f0231f1c0c11?showWhenStarting=1#) with the password Passw0rd!.

