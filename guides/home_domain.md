# Home domain

In order to make obvious that DigitalBits Network Node belongs to the domain following steps are required.

1. Create file [digitalbits.toml](https://github.com/xdbfoundation/docs/tree/master/guides/concepts/digitalbits-toml.md)

2. Create account for the keypair specified in config file of your node. 
For this, you need to make a transaction of type 'Create account' with destinaton
being public key of your node keypair and at list minimum ammount needed to be on the account balance (20XDB for new account. See [minimum balance](https://github.com/xdbfoundation/docs/tree/master/guides/concepts/fees.md#minimum-account-balance)). As each transaction results in [fee](https://github.com/xdbfoundation/docs/tree/master/guides/concepts/fees.md), fund enough to be able to accomplish next step.

3. Make a transaction of type 'Set options' and set option 'home_domain' to the domain, where your digitalbits.toml file is located. For example 'mycompany.com'

You may use [Laboratory](https://laboratory.livenet.digitalbits.io/#txbuilder) to build and submit transactions to the network.

# Example

Suppose we already have an account on DigitalBits Livenet Network with private key `SBUUJM5Q72XFGWF2SXHKJYTH5PBYQVHAWCPFRWB5AXNKGSBC4WM27CMZ` and public key `GDZDT2AZ65PALRS5COUBRRXPPESU3VB3PTP3BGRQSFI6JTCD6SKWMGWW` and domain `https://example.digitalbits.io`. We started three validator nodes. 


## Node Configs 


### Node in Ireland 
Config file of the first node (public key `GBV3UQVVG5TSO6KYHGNDGK4HAEUW7F5VGJSOZD27OFGRKYBHWIHDZDJD` and url `https://ireland.example.digitalbits.io`) contains following line:

```
# ...
NODE_SEED="SAVFRRS7BGFGVDJEZBDNSXNLP7S7XJEXRDXKDFA4ARQCXVF4H4ZC4MSL self"
[[HOME_DOMAINS]]
HOME_DOMAIN="example.digitalbits.io"
QUALITY="HIGH"
[HISTORY.local]
get="curl -sf https://example.digitalbits.io/livenet/germany/{0} -o {1}"
put="aws s3 cp {0} s3://livenet-mycompany-s3bucket/livenet/germany/{1}"
# ...
``` 

### Node in France

Second node (public key `GDSKRPNLL3HA7CUURJOEZJNJY43TGPTGMERKU7B4MJH2QDYIGULHFS62` and url `https://france.example.digitalbits.io`)

```
# ...
NODE_SEED="SARSCNPJ3ER7MMKMTDYFNE2DKZTRX7I5M6RJ6BSGFHMSJ5OC6VMPHIAZ self"
[[HOME_DOMAINS]]
HOME_DOMAIN="example.digitalbits.io"
QUALITY="HIGH"
[HISTORY.local]
get="curl -sf https://example.digitalbits.io/livenet/france/{0} -o {1}"
put="aws s3 cp {0} s3://livenet-mycompany-s3bucket/france/2/{1}"
# ...
``` 

### Node in Germany

Third node (public key `GAOWJ3P2OQGK56Y6V7UIQEPVOGOWFKOI5JJ5AGNNVRPC6BZLNUFDCANF`and url `https://germany.example.digitalbits.io`)

```
# ...
NODE_SEED="SD2BKLRQ4ZMVWBBN3J4MSBESLGOZI7BQGUZCMUHZES3SUAKT77PK6SFN self"
[[HOME_DOMAINS]]
HOME_DOMAIN="example.digitalbits.io"
QUALITY="HIGH"
[HISTORY.local]
get="curl -sf https://example.digitalbits.io/livenet/3/{0} -o {1}"
put="aws s3 cp {0} s3://livenet-mycompany-s3bucket/livenet/3/{1}"
# ...
``` 

## digitalbits.toml

After we ran our validator nodes, we need to upload [digitalbits.toml](https://github.com/xdbfoundation/docs/tree/master/guides/concepts/digitalbits-toml.md) to out domain at path `<domain>./well-known/digitalbits.toml`

In our case, home domain is `example.digitalbits.io`, so the `digitalbits.toml` file should be located at URL `https://example.digitalbits.io/.well-known/digitalbits.toml`  command.

    curl https://example.digitalbits.io/.well-known/digitalbits.toml

will result in the following content of the `digitalbits.toml file`


    VERSION="2.0.0"

    NETWORK_PASSPHRASE="LiveNet Global DigitalBits Network ; February 2021"
    FRONTIER_URL="https://frontier.example.digitalbits.io/"
    ACCOUNTS=[
        "GBV3UQVVG5TSO6KYHGNDGK4HAEUW7F5VGJSOZD27OFGRKYBHWIHDZDJD",
        "GDSKRPNLL3HA7CUURJOEZJNJY43TGPTGMERKU7B4MJH2QDYIGULHFS62",
        "GAOWJ3P2OQGK56Y6V7UIQEPVOGOWFKOI5JJ5AGNNVRPC6BZLNUFDCANF"
        ]

    [DOCUMENTATION]
    ORG_NAME="DigitalBits Example"
    ORG_URL="https://example.digitalbits.io"

    [[VALIDATORS]]
    ALIAS="example-digitalbits-irl"
    DISPLAY_NAME="DigitalBits Example Ireland"
    HOST="ireland.example.digitalbits.io:11625"
    PUBLIC_KEY="GBV3UQVVG5TSO6KYHGNDGK4HAEUW7F5VGJSOZD27OFGRKYBHWIHDZDJD"
    HISTORY="https://example.digitalbits.io/livenet/ireland/"


    [[VALIDATORS]]
    ALIAS="example-digitalbits-fra"
    DISPLAY_NAME="DigitalBits Example France"
    HOST="france.example.digitalbits.io:11625"
    PUBLIC_KEY="GDSKRPNLL3HA7CUURJOEZJNJY43TGPTGMERKU7B4MJH2QDYIGULHFS62"
    HISTORY="https://example.digitalbits.io/livenet/fracce/"

    [[VALIDATORS]]
    ALIAS="example-digitalbits-deu"
    DISPLAY_NAME="DigitalBits Example Germany"
    HOST="germany.example.digitalbits.io:11625"
    PUBLIC_KEY="GAOWJ3P2OQGK56Y6V7UIQEPVOGOWFKOI5JJ5AGNNVRPC6BZLNUFDCANF"
    HISTORY="https://history.livenet.digitalbits.io/germany/"


## Transactions 

### Node Account

For each on our validator node (`GBV3UQVVG5TSO6KYHGNDGK4HAEUW7F5VGJSOZD27OFGRKYBHWIHDZDJD`, `GDSKRPNLL3HA7CUURJOEZJNJY43TGPTGMERKU7B4MJH2QDYIGULHFS62`, `GAOWJ3P2OQGK56Y6V7UIQEPVOGOWFKOI5JJ5AGNNVRPC6BZLNUFDCANF`) we need to create an account (fund them) and set option `home_domain` for that account. 

First, we create account for each of our node, by making an transaction of type `Create Account` from our existing account, `GDZDT2AZ65PALRS5COUBRRXPPESU3VB3PTP3BGRQSFI6JTCD6SKWMGWW`. 


### Create Account Transaction 

To do this, we create and submit transaction at [DigitalBits Laboratory](https://laboratory.livenet.digitalbits.io/#txbuilder). 

Transaction Type:  `Transaction`

Source Account: `GDZDT2AZ65PALRS5COUBRRXPPESU3VB3PTP3BGRQSFI6JTCD6SKWMGWW`

Transaction Sequence Number: `4658068126171137` (web get this number by pressing `Fetch next sequence number for account starting with "GDZDT2AZ65"`)

Base Fee: `100 nibbs`

Memo: `None`

Time Bounds: (optional) `1624563492` (Press `Set to 5 minutes from now` button)

Operation Type: `Create Account`

Destination: `GBV3UQVVG5TSO6KYHGNDGK4HAEUW7F5VGJSOZD27OFGRKYBHWIHDZDJD`

Strating Balance: 1000XDB (20XDB is the minimal ammount for `Create Account` operation).

Then we press `Sign in Transaction Signer` button. 

Add Signer: `SBUUJM5Q72XFGWF2SXHKJYTH5PBYQVHAWCPFRWB5AXNKGSBC4WM27CMZ` (out existing account private key)

Press `Submit in Transaction Submitter` and then `Submit Transaction` button. If transaction succeeds, we created an account for our first validating node. We need to do this also for our second and third validating node.


### Set Options 'home_domain' Transaction

Then we need to set home domain on that account, by making an transaction of type `Set options` at [DigitalBits Laboratory](https://laboratory.livenet.digitalbits.io)


Source Account: `GBV3UQVVG5TSO6KYHGNDGK4HAEUW7F5VGJSOZD27OFGRKYBHWIHDZDJD` (our first validating node public key)

Transaction Sequence Number ...

Base Fee ...

Memo ...

Time Bounds ...

Operation Type: `Set Options`

Home Domain: `example.digitalbits.io`

Press `Sign in Transaction Signer`

Add Signer: `SAVFRRS7BGFGVDJEZBDNSXNLP7S7XJEXRDXKDFA4ARQCXVF4H4ZC4MSL` (our first validating node private key)

Than submit transaction to the DigitalBits Livenet Network. Do this to the rest of the nodes of your domain. 

