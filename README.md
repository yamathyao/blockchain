# blockchain

### hyperledger fabric 

* 搭建基础环境

  docker, docker compose, java, git等
  
* 文件夹含义

    `/opt/hyperledger/bin` - 工具  
    `/opt/hyperledger/channel-artifacts` - 区块    
    `/opt/hyperledger/config` - 配置    
    `/opt/hyperledger/crypto-config` - 生成的证书    
    `/opt/hyperledger/docker-compose` - docker启动文件    
  
* 证书配置

  - 配置 crypto-config.yaml  
  `/opt/hyperledger/config/crypto-config.yaml`  
  - 生成证书  
  `./cryptogen generate --config=/opt/hyperledger/config/crypto-config.yaml --output /opt/hyperledger/crypto-config`  
  - 配置 hosts  
  orderer.fabric.com  
  peer0.org1.fabric.com  
  peer0.org2.fabric.com
  
* 生成创始块

    - 配置 configtx.yaml  
      `/opt/hyperledger/config/configtx.yaml`  
    - 生成创始块  
    `./configtxgen -profile TwoOrgsOrdererGenesis -channelID system-channel -configPath /opt/hyperledger/config -outputBlock /opt/hyperledger/channel-artifacts/genesis.block`  
    
* 创建通道事务

    `./configtxgen -profile TwoOrgsChannel -outputCreateChannelTx /opt/hyperledger/channel-artifacts/channel1.tx -channelID channel1 -configPath /opt/hyperledger/config`
    
* 为组织更新锚点

    `./configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate /opt/hyperledger/channel-artifacts/Org1MSPanchors
    .tx -channelID channel1 -asOrg Org1MSP -configPath /opt/hyperledger/config`  
    `./configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate /opt/hyperledger/channel-artifacts/Org2MSPanchors
    .tx -channelID channel1 -asOrg Org2MSP -configPath /opt/hyperledger/config`  
        
* 启动 order

    `/opt/hyperledger/docker-compose/docker-compose-orderer.yaml`  
    