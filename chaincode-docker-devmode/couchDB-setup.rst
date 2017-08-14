Using Couch DB as state database
==============

By default peer uses goleveldb as its default embeded key value state database.
However if we want to perform complex queries on the state content (non-key queries) we can configure the peer to use couchDB as its state database.


.. note:: Make sure that the required docker images are downloaded. If not then follow the stepd in: 
[a link](https://github.com/nitesh7sid/chaincode-dev-mode/edit/master/chaincode-docker-devmode/Readme.rst) 
          
          
Terminal 1 - Start the couchDB container
------------------------------

.. code:: bash

    docker-compose -f docker-compose-simple.yaml up

The above starts the network with the ``SingleSampleMSPSolo`` orderer profile and
launches the peer in "dev mode".  It also launches two additional containers -
one for the chaincode environment and a CLI to interact with the chaincode.  The
commands for create and join channel are embedded in the CLI container, so we
can jump immediately to the chaincode calls.

Terminal 2 - Build & start the chaincode
----------------------------------------

.. code:: bash

  docker exec -it chaincode bash

You should see the following:

.. code:: bash

  root@d2629980e76b:/opt/gopath/src/chaincode#

Now, compile your chaincode:

.. code:: bash

  cd helloWorld
  go build

Now run the chaincode:

.. code:: bash

  CORE_PEER_ADDRESS=peer:7051 CORE_CHAINCODE_ID_NAME=mycc:0 ./helloWorld

The chaincode is started with peer and chaincode logs indicating successful registration with the peer.
Note that at this stage the chaincode is not associated with any channel. This is done in subsequent steps
using the ``instantiate`` command.

Terminal 3 - Use the chaincode
------------------------------

Even though you are in ``--peer-chaincodedev`` mode, you still have to install the
chaincode so the life-cycle system chaincode can go through its checks normally.
This requirement may be removed in future when in ``--peer-chaincodedev`` mode.

We'll leverage the CLI container to drive these calls.

.. code:: bash

  docker exec -it cli bash

.. code:: bash

  peer chaincode install -p chaincodedev/chaincode/chaincode_example02 -n mycc -v 0
  peer chaincode instantiate -n mycc -v 0 -c '{"Args":["init"]}' -C myc

Now issue an invoke to move initialize the KV store.

.. code:: bash

  peer chaincode invoke -n mycc -c '{"Args":["writeFunc1","key","hello world"]}' -C myc

Finally, query ``key``.  We should see a value of ``hello world``.

.. code:: bash

  peer chaincode query -n mycc -c '{"Args":["readFunc1","key"]}' -C myc
