# API

## Interface

A single endpoint for the blockchain naming system is defined:  `GET /:versionString/names/:fullyQualifiedName`

* `:versionString` is the resolver API version.  For now, this is `1.0`.
* `:fullyQualifiedName` is the fully-qualified name to query.  This includes any/all indications of things like which system the name lives in, which blockchain the name is registered on, which namespace it lives in, and so on.

Examples:

* `GET /1.0/names/judecn.id.bsk` resolves `judecn.id` in Blockstack’s virtualchain.
* `GET /1.0/names/nickjohnson.eth.ens` resolves `nickjohnson.eth` using ENS.

### Fully-Qualified Names

Each name should end in a '.' and a suffix that identifies the service that can resolve
the name.  This is similar in function to the method ID in a DID.  Anyone can
claim an identifier, provided that it is unique.

Identifiers in use today:

* `.bsk`: for Blockstack-hosted names
* `.ens`: for ENS-hosted names
* ... (insert yours here)

## Return Values

We are still waiting on a well-defined schema for a DDO, but the jist of the return value is that it is a compound object:

* the version string (matches `:versionString` in the request)
* the DDO identified by the fully-qualified name
* a catch-all `supplementary` object that provides driver- and service-specific hints to the client.

The `supplementary` field is for forward compatibility with future systems.  This is a field a driver can use to return something service-specific to clients (for example, Blockstack and ENS might want to give back the relevant transaction ID that created the DID).  If it is discovered that each driver returns the same types of data in their `supplementary` fields, then we will standardize the response in a future version of this specification.

Schema (NOTE: missing precise DDO fields)

```
    {
       'type': 'object',
       'properties': {
          'version': {
             'type': 'string',
             'pattern': '^1\.0$',
          },
          'ddo': {
             'type': 'object',
             'properties': {
                '...'
             }
             'required': [
                '@context',
                'id',
                'signature',
             ],
          },
          'supplementary': {
             'type': 'object',
             'additionalProperties': true,
          },
       },
       'additionalProperties': false,
    }           
```

Examples (DDO fields gleaned from https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-fall2016/blob/master/did-spec-wd03.md)

```
    {
       'version': '1.0',
       'ddo': {
           '@context': '...',
           'id': 'did:sov:21tDAKCERh95uGgKbJNHYp',
           'equiv-id': [
              'did:sov:33ad7beb1abc4a26b89246',
              'did:sov:33ad7beb1abc4a26b89246',
              'did:bstk:judecn.id',
              'did:ens:judecn.eth',
           ],
           'verkey': 'MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCABMC',
           'control': [
              'self',
              'did:bstk:judecn.id',
           ],
           'service': {
              'dif': 'https://dif.microsoft.com',
              'blockstack': 'https://explorer.blockstack.org',
              'ens': 'https://etherscan.io/enslookup',
              'openid': 'https://vicci.org/id',
           },
           'type': 'https://schema.org/Person',
           'creator': 'did:bstk:judecn.id',
           'created': '2016-10-10T17:00:00Z',
           'updated': '2017-03-14T18:00:30Z',
           'signature': {
              'type': 'LinkedDataSignature2015',
              'created': '2016-02-08T16:02:20Z',
              'creator': 'did:76d0cdb7-9c75-4be5-8e5a-e2d7a35ce907/keys/1',
              'signatureValue': 'QNB13Y7Q9oLlDLL6AHyL31OE5fLji9DwJSA8qnv81oRaKonij8m+Jv4XdiEYvJ97iRlzKU/92/0LafSL5JftEgl960DLcbqMFxOtbAmFOIMa7eDcrgTL5ytXeYCYKLjHQG3s8a3UKDKRuEK54qK1G5hGKGoLgAVa6xgcDLjW7M19PEJV/c3HpGA7Eez6VFMoTt4yESjZvOXC97xN3KpshOx2HT/btgUbo0XjA1Oi0QHdgrLcUsQGt6w23RjeSToalrsA1G69OFeN2OiQrz9Jb4561hvKLSyWObwRmS6n5Vgr5xkvUm6MONRq0Vg33kXevoVM64KTBkISul61tzjn4w==',
           },
        },
        'supplementary': {
           'whois': {
              "block_preordered_at": 373622, 
              "block_renewed_at": 373622, 
              "expire_block": 489247, 
              "has_zonefile": true, 
              "last_transaction_height": 422657, 
              "last_transaction_id": "5e4c0a42874ac59a9e21d351ec8b06971231ed2c02e0bac61b7dcc8ee8cb1ad2", 
              "owner_address": "16EMaNw3pkn3v6f2BgnSSs53zAKH4Q8YJg", 
              "owner_script": "76a914395f3643cea07ec4eec73b4d9a973dcce56b9bf188ac", 
              "zonefile_hash": "4d718e536b57c2dcc556cf2cdbadf1b6647ead80"
           },
           'zonefile': "$ORIGIN judecn.id\n$TTL 3600\npubkey TXT \"pubkey:data:04cabba0b5b9a871dbaa11c044066e281c5feb57243c7d2a452f06a0d708613a46ced59f9f806e601b3353931d1e4a98d7040127f31016311050bedc0d4f1f62ff\"\n_file URI 10 1 \"file:///home/jude/.blockstack/storage-disk/mutable/judecn.id\"\n_https._tcp URI 10 1 \"https://blockstack.s3.amazonaws.com/judecn.id\"\n_http._tcp URI 10 1 \"http://node.blockstack.org:6264/RPC2#judecn.id\"\n_dht._udp URI 10 1 \"dht+udp://fc4d9c1481a6349fe99f0e3dd7261d67b23dadc5\"\n"
       },
    }       
```