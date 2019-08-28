********************
API Reference Node
********************

Nodes are the backend of Incubed. Each node serves RPC requests to Incubed clients. The node itself runs like a proxy for an Ethereum client (Geth, Parity, etc.), but instead of simply passing the raw response, it will add the required proof needed by the client to verify the response.

To run such a node, you need to have an Ethereum client running where you want to forward the request to. At the moment, the minimum requirement is that this client needs to support ``eth_getProof`` (see http://eips.ethereum.org/EIPS/eip-1186).

Command-line Arguments
####################

--autoRegistry-capabilities-multiChain   If true, this node is able to deliver multiple chains.
--autoRegistry-capabilities-proof        If true, this node is able to deliver proofs.
--autoRegistry-capacity                  Max number of parallel requests.
--autoRegistry-deposit                   The deposit you want to store.
--autoRegistry-depositUnit               Unit of the deposit value.
--autoRegistry-url                       The public URL to reach this node.
--cache                                  Cache Merkle tries.
--chain                                  ChainId.
--clientKeys                             A comma-separated list of client keys to use for simulating clients for the watchdog.
--db-database                            Name of the database.
--db-host                                Db-host (default = local host).
--db-password                            Password for db-access.
--db-user                                Username for the db.
--defaultChain                           The default chainId in case the request does not contain one.
--freeScore                              The score for requests without a valid signature.
--handler                                The implementation used to handle the calls.
--help                                   Output usage information.
--id                                     An identifier used in log files for reading the configuration from the database.
--ipfsUrl                                The URL of the IPFS client.
--logging-colors                         If true, colors will be used.
--logging-file                           The path to the log file.
--logging-host                           The host for custom logging.
--logging-level                          Log level.
--logging-name                           The name of the provider.
--logging-type                           The module of the provider.
--maxThreads                             The maximal number of threads running parallel to the processes.
--minBlockHeight                         The minimal block height needed to sign.
--persistentFile                         The file name of the file keeping track of the last handled blockNumber.
--privateKey                             The private key used to sign blockhashes. This can be either a 0x-prefixed string with the raw private key or the path to a key file.
--privateKeyPassphrase                   The password used to decrypt the private key.
--profile-comment                        Comments for the node.
--profile-icon                           URL to an icon or logo of a company offering this node.
--profile-name                           Name of the node or company.
--profile-noStats                        If active, the stats will not be shown (default: false).
--profile-url                            URL of the website of the company.
--registry                               The address of the server registry used to update the NodeList.
--registryRPC                            The URL of the client in case the registry is not on the same chain.
--rpcUrl                                 The URL of the client.
--startBlock                             BlockNumber to start watching the registry.
--timeout                                Number of milliseconds needed to wait before a request times out.
--version                                Output of the version number.
--watchInterval                          The number of seconds before a new event.
--watchdogInterval                       Average time between sending requests to the same node. 0 turns it off (default).

Registering Your Own Incubed Node
##########################

If you want to participate in this network and register a node, you need to send a transaction to the registry contract, calling `registerServer(string _url, uint _props)`.

To run an Incubed node, you simply use docker-compose:

.. code-block:: yaml

        version: '2'
        services:
        incubed-server:
            image: slockit/in3-server:latest
            volumes:
            - $PWD/keys:/secure                                     # Directory where the private key is stored.
            ports:
            - 8500:8500/tcp                                         # Open the port 8500 to be accessed by the public.
            command:
            - --privateKey=/secure/myKey.json                       # Internal path to the key.
            - --privateKeyPassphrase=dummy                          # Passphrase to unlock the key.
            - --chain=0x1                                           # Chain (Kovan).
            - --rpcUrl=http://incubed-parity:8545                   # URL of the Kovan client.
            - --registry=0xFdb0eA8AB08212A1fFfDB35aFacf37C3857083ca # URL of the Incubed registry. 
            - --autoRegistry-url=http://in3.server:8500             # Check or register this node for this URL.
            - --autoRegistry-deposit=2                              # Deposit to use when registering.

        incubed-parity:
            image: parity:latest                                    # Parity image with the proof function implemented.
            command:
            - --auto-update=none                                    # Do not automatically update the client.
            - --pruning=archive 
            - --pruning-memory=30000                                # Limit storage.
            - --jsonrpc-experimental                                # Currently still needed until EIP 1186 is finalized.
