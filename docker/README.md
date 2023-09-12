# BlockScout Docker Integration

## 编译容器
make build

## 使用容器
```text
version: '3.8'

services:
  db:
    image: postgres:latest
    restart: always
    container_name: 'postgres'
    command: postgres -c 'max_connections=200'
    environment:
        POSTGRES_PASSWORD: 'polaris'
        POSTGRES_USER: 'postgres'
    ports:
      - 5432:5432
    #network_mode: host
    volumes:
      - ./blockscout-db-data:/var/lib/postgresql/data/

  smart-contract-verifier:
    image: yuhuajing/smart-contract-verifier:latest
    restart: always
    container_name: 'smart-contract-verifier'
    ports:
      - 8050:8050
    #network_mode: host
    environment:
      - SMART_CONTRACT_VERIFIER__SOLIDITY__ENABLED=true
      - SMART_CONTRACT_VERIFIER__VYPER__ENABLED=false
      - SMART_CONTRACT_VERIFIER__SOURCIFY__ENABLED=false
      - SMART_CONTRACT_VERIFIER__METRICS__ENABLED=false    

  blockscout:
    depends_on:
      - db
      - smart-contract-verifier
    image: blockscout/blockscout:latest
   # restart: always
    stop_grace_period: 5m
    container_name: 'blockscout'
    command: bash -c "bin/blockscout eval \"Elixir.Explorer.ReleaseTasks.create_and_migrate()\" && bin/blockscout start"
    extra_hosts:
      - 'host.docker.internal:host-gateway'
 #   network_mode: host
    environment:
      - MIX_ENV=prod
      - ETHEREUM_JSONRPC_VARIANT=geth
      - CHAIN_ID=12345
      - ETHEREUM_JSONRPC_HTTP_URL=http://172.21.87.231:8545
      - ETHEREUM_JSONRPC_TRACE_URL=http://172.21.87.231:8545
      - ETHEREUM_JSONRPC_WS_URL=ws://172.21.87.231:8546
        #- FIRST_BLOCK=?xx
        #- TRACE_FIRST_BLOCK=?xx
      - SECRET_KEY_BASE="l4Z2o45D9Cvz7lezp30vxu4mCXqeKzXozjJjioYzWBlXM/NHEAu9/2OyWTIM0+1Y"
      - SUBNETWORK=Orbites
      - COIN=ORB
      - COIN_NAME=ORB
      - DATABASE_URL=postgresql://postgres:polaris@host.docker.internal:5432/blockscout
      - ECTO_USE_SSL=false
      - BLOCKSCOUT_PROTOCOL="http"
      - PORT=4000
      - INDEXER_DISABLE_BLOCK_REWARD_FETCHER=true
      - HIDE_BLOCK_MINER=true
        #  - INDEXER_DISABLE_INTERNAL_TRANSACTIONS_FETCHER=true #disable internal tx debug
        #- ETHEREUM_JSONRPC_DISABLE_ARCHIVE_BALANCES=true # disable getBalance from before blocks
        # - DISABLE_EXCHANGE_RATES=true
      - POOL_SIZE=50
      - POOL_SIZE_API=50
      - ACCOUNT_POOL_SIZE=10
      - HEART_BEAT_TIMEOUT=600
      - INDEXER_EMPTY_BLOCKS_SANITIZER_BATCH_SIZE=500
      - INDEXER_MEMORY_LIMIT="10GB"
      #- FETCH_REWARDS_WAY="manual"
      - MICROSERVICE_SC_VERIFIER_ENABLED=true
      - MICROSERVICE_SC_VERIFIER_URL=http://host.docker.internal:8050
    ports:
      - 4000:4000
    volumes:
      - ./logs/:/app/logs/
```