https://geth.ethereum.org/docs/getting-started/installing-geth
https://geth.ethereum.org/docs/getting-started/consensus-clients
https://docs.prylabs.network/docs/install/install-with-script

nohup geth --mainnet --port 20303 --http --ws --ws.port 8002 --http.api eth,net,engine,admin,web3 --http.port 8000 --authrpc.jwtsecret=/app/jwt.hex > /dev/null 2>&1&

nohup ./prysm.sh beacon-chain --execution-endpoint=http://localhost:8551 --mainnet --jwt-secret=/app/jwt.hex --checkpoint-sync-url=https://beaconstate.info --genesis-beacon-api-url=https://beaconstate.info > /dev/null 2>&1&

# base node

docker run --env-file .env.mainnet -e OP_NODE_L2_ENGINE_RPC=ws://localhost:8551 -e OP_NODE_RPC_PORT=7545 ghcr.io/base-org/node:latest
