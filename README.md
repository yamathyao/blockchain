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

    `./configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate /opt/hyperledger/channel-artifacts/Org1MSPanchors.tx -channelID channel1 -asOrg Org1MSP -configPath /opt/hyperledger/config`  
    `./configtxgen -profile TwoOrgsChannel -outputAnchorPeersUpdate /opt/hyperledger/channel-artifacts/Org2MSPanchors.tx -channelID channel1 -asOrg Org2MSP -configPath /opt/hyperledger/config`  
        
* 启动 order

    `/opt/hyperledger/docker-compose/docker-compose-orderer.yaml`  
    
* 启动 peer 和 cli

    `/opt/hyperledger/docker-compose/docker-compose-peer-org1.yaml`  
    `/opt/hyperledger/docker-compose/docker-compose-peer-org2.yaml`  
    
* 创建应用通道

    进入 org1-cli 容器内部  
    `docker exec -it org1-cli bash`  
    创建 channel1    
    `peer channel create -o orderer.fabric.com:7050 -c channel1 -f /opt/hyperledger/channel-artifacts/channel1.tx --tls true --cafile /opt/hyperledger/crypto-config/ordererOrganizations/fabric.com/orderers/orderer.fabric.com/msp/tlscacerts/tlsca.fabric.com-cert.pem`    
    org1 加入 channel1  
    `peer channel join -b channel1.block`  
    把 channel1.block scp 到 peer.org2.fabric.com 的主机目录下  
    `/opt/hyperledger/crypto-config/peerOrganizations/org2.fabric.com/peers/peer0.org2.fabric.com`  
    进入 org2-cli 容器内部  
    `docker exec -it org2-cli bash`  
    org2 加入 channel1  
    `peer channel join -b channel1.block`  
    
* 打包 chaincode

    `git clone http://192.168.7.197:8083/Architecture/geex-spring-fabric-chaincode.git`    
    org1-cli 容器内部，打包 chaincode tar 包  
    `peer lifecycle chaincode package geex-spring-fabric-chaincode.tar.gz --path geex-spring-fabric-chaincode/ --lang java --label geex-spring-fabric-chaincode`  
    
* 安装 chaincode

    peer0.org1.fabric.com 安装 chaincode
    `peer lifecycle chaincode install geex-spring-fabric-chaincode.tar.gz`  
    检查 chaincode 安装情况，获得 Package ID  
    `peer lifecycle chaincode queryinstalled`    
    当前组织同意 chaincode  
    `peer lifecycle chaincode approveformyorg -o orderer.fabric.com:7050 --ordererTLSHostnameOverride orderer.fabric.com --tls true --cafile /opt/hyperledger/crypto-config/ordererOrganizations/fabric.com/orderers/orderer.fabric.com/msp/tlscacerts/tlsca.fabric.com-cert.pem --channelID channel1 --name geex-spring-fabric-chaincode --version 1.0.0 --init-required --package-id geex-spring-fabric-chaincode:417b11db9712785a7fdaa8372f9357265aee1a35726e84f082c66bf1d99af3f4 --sequence 1 --waitForEvent`   
    检查合约定义，得到结果 Org1MSP: true, Org2MSP: false  
    `peer lifecycle chaincode checkcommitreadiness --channelID channel1 --name geex-spring-fabric-chaincode --version 1.0.0 --sequence 1 --output json --init-required`     
    
    peer0.org2.fabric.com 安装 chaincode，步骤同 peer0.org1.fabric.com  
    得到结果 Org1MSP: true, Org2MSP: true  
    
    提交链码  
    `peer lifecycle chaincode commit -o orderer.fabric.com:7050 --tls true --cafile /opt/hyperledger/crypto-config/ordererOrganizations/fabric.com/msp/tlscacerts/tlsca.fabric.com-cert.pem --channelID channel1 --name geex-spring-fabric-chaincode --peerAddresses peer0.org1.fabric.com:7051 --tlsRootCertFiles /opt/hyperledger/crypto-config/peerOrganizations/org1.fabric.com/peers/peer0.org1.fabric.com/tls/ca.crt --peerAddresses peer0.org2.fabric.com:7051 --tlsRootCertFiles /opt/hyperledger/crypto-config/peerOrganizations/org2.fabric.com/peers/peer0.org2.fabric.com/tls/ca.crt --version 1.0.0 --sequence 1 --init-required`  
    查看提交的合约  
    `peer lifecycle chaincode querycommitted --channelID channel1 --name geex-spring-fabric-chaincode`
    
* 操作链码

    初始化链码   
    `peer chaincode invoke -o orderer.fabric.com:7050 --ordererTLSHostnameOverride orderer.fabric.com --tls --cafile /opt/hyperledger/crypto-config/ordererOrganizations/fabric.com/orderers/orderer.fabric.com/msp/tlscacerts/tlsca.fabric.com-cert.pem --channelID channel1 --name geex-spring-fabric-chaincode --peerAddresses peer0.org1.fabric.com:7051 --tlsRootCertFiles /opt/hyperledger/crypto-config/peerOrganizations/org1.fabric.com/peers/peer0.org1.fabric.com/tls/ca.crt --peerAddresses peer0.org2.fabric.com:7051 --tlsRootCertFiles /opt/hyperledger/crypto-config/peerOrganizations/org2.fabric.com/peers/peer0.org2.fabric.com/tls/ca.crt --isInit -c '{"function":"init","Args":["a","123"]}'`  
    查询交易     
    `peer chaincode query -C channel1 -n geex-spring-fabric-chaincode -c '{"function":"query","Args":["a"]}'`    
     
    