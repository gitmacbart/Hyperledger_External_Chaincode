# Hyperledger_External_Chaincode Lab
## Build the Network
```
cd hlf_network

cp -R config /tmp/hyperledger/config/
cp -R sampleBuilder/ /tmp/hyperledger/sampleBuilder/
```
```
docker-compose up -d rca-org1 rca-org2 ca-tls
 ./allCAnReg.sh
 ./enrollAllOrgs.sh
docker-compose up -d peer1-org1 peer1-org2 orderer1-org1 orderer1-org2 couchdb0 couchdb1
```
```
source term-org1
configtxgen -profile ${CHANNEL_PROFILE} -configPath ${PWD} -outputBlock ${CHANNEL_NAME}.block -channelID ${CHANNEL_NAME}

osnadmin channel join --channelID ${CHANNEL_NAME} --config-block ${CHANNEL_NAME}.block -o $CORE_ORDERER_ADDRESS --ca-file $ORDERER_CA --client-cert $ADMIN_TLS_CERTFILE  --client-key $ADMIN_TLS_KEYFILE
peer channel join -b ${CHANNEL_NAME}.block

source term-org2
osnadmin channel join --channelID ${CHANNEL_NAME} --config-block ${CHANNEL_NAME}.block -o $CORE_ORDERER_ADDRESS --ca-file $ORDERER_CA --client-cert $ADMIN_TLS_CERTFILE  --client-key $ADMIN_TLS_KEYFILE
peer channel join -b ${CHANNEL_NAME}.block

## Install the Chaincode sacc

```
## Install the Chaincode
```
cd ../external_sacc
tar cfz code.tar.gz connection.json
tar cfz sacc_external.tgz metadata.json code.tar.gz

source term-org1
peer lifecycle chaincode install ../external_sacc/sacc_external.tgz

source term-org2
peer lifecycle chaincode install ../external_sacc/sacc_external.tgz

source term-org1
peer lifecycle chaincode approveformyorg --tls --cafile $ORDERER_CA -o localhost:7050 --channelID channel1 --name sacc --version 1 --init-required --sequence 1 --waitForEvent --package-id xxxxx (refer from previous install command)

source term-org2
peer lifecycle chaincode approveformyorg --tls --cafile $ORDERER_CA -o localhost:8050 --channelID channel1 --name sacc --version 1 --init-required --sequence 1 --waitForEvent --package-id xxxxx

source term-org1
peer lifecycle chaincode commit --tls --cafile $ORDERER_CA -o localhost:7050 --peerAddresses $CORE_PEER_ADDRESS --tlsRootCertFiles $CORE_PEER_TLS_ROOTCERT_FILE --peerAddresses localhost:8051 --tlsRootCertFiles /tmp/hyperledger/org2/peer1/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem --channelID channel1 --name sacc --version 1 --sequence 1 --init-required

## Build & deploy the Chaincode container sacc

```
## Build & deploy the Chaincode container
```
cd ../external_sacc
GO111MODULE=on go mod vendor
docker build -t hyperledger/sacc-ext .

cd ../hlf_network
docker-compose up -d sacc-ext

## Invoke and Query the external chaincode sacc

```
## Invoke and Query the external chaincode
```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer1-org1 --tls true --cafile $ORDERER_CA --peerAddresses localhost:7051 --tlsRootCertFiles /tmp/hyperledger/org1/peer1/assets/tls-ca/tls-ca-cert.pem --peerAddresses localhost:8051 --tlsRootCertFiles /tmp/hyperledger/org2/peer1/assets/tls-ca/tls-ca-cert.pem --channelID channel1 --name sacc --isInit -c '{"Args":["name","doe"]}'

peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer1-org1 --tls true --cafile $ORDERER_CA --peerAddresses localhost:7051 --tlsRootCertFiles /tmp/hyperledger/org1/peer1/assets/tls-ca/tls-ca-cert.pem --peerAddresses localhost:8051 --tlsRootCertFiles /tmp/hyperledger/org2/peer1/assets/tls-ca/tls-ca-cert.pem --channelID channel1 --name sacc -c '{"Args":["set","name","JohnDoe"]}'

peer chaincode query -C channel1 -n sacc -c '{"Args":["get","name"]}'

```
# Install the Chaincode Flightchain
```
cd ../external_flightchain
tar cfz code.tar.gz connection.json
tar cfz fchain_external.tgz metadata.json code.tar.gz

source term-org1
peer lifecycle chaincode install ../external_flightchain/fchain_external.tgz

source term-org2
peer lifecycle chaincode install ../external_flightchain/fchain_external.tgz

source term-org1
peer lifecycle chaincode approveformyorg --tls --cafile $ORDERER_CA -o localhost:7050 --channelID channel1 --name fchain --version 1 --init-required --sequence 1 --waitForEvent --package-id xxxxx (refer from previous install command)

source term-org2
peer lifecycle chaincode approveformyorg --tls --cafile $ORDERER_CA -o localhost:8050 --channelID channel1 --name fchain --version 1 --init-required --sequence 1 --waitForEvent --package-id xxxxx

source term-org1
peer lifecycle chaincode commit --tls --cafile $ORDERER_CA -o localhost:7050 --peerAddresses $CORE_PEER_ADDRESS --tlsRootCertFiles $CORE_PEER_TLS_ROOTCERT_FILE --peerAddresses localhost:8051 --tlsRootCertFiles /tmp/hyperledger/org2/peer1/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem --channelID channel1 --name fchain --version 1 --sequence 1 --init-required
```
## Build & deploy the Chaincode container flightchain
```
docker image pull sitapseacr.azurecr.io/sitapseacr/flightchain3/flightchain.smartcontract-fly2plan-fabric2:latest

cd ../hlf_network
docker-compose up -d fchain-ext
```
## Invoke and Query the external chaincode
```
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer1-org1 --tls true --cafile $ORDERER_CA --peerAddresses localhost:7051 --tlsRootCertFiles /tmp/hyperledger/org1/peer1/assets/tls-ca/tls-ca-cert.pem --peerAddresses localhost:8051 --tlsRootCertFiles /tmp/hyperledger/org2/peer1/assets/tls-ca/tls-ca-cert.pem --channelID channel1 --name fchain --isInit -c '{"Args":["FlightChainContract:initLedger"]}'

peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer1-org1 --tls true --cafile $ORDERER_CA --peerAddresses localhost:7051 --tlsRootCertFiles /tmp/hyperledger/org1/peer1/assets/tls-ca/tls-ca-cert.pem --peerAddresses localhost:8051 --tlsRootCertFiles /tmp/hyperledger/org2/peer1/assets/tls-ca/tls-ca-cert.pem --channelID channel1 --name fchain -c '{"Args":["createFlight", "{\"operatingAirline\": {\"iataCode\": \"EI\"},\"departureAirport\": \"DUB\",\"arrivalAirport\": \"JFK\",\"flightNumber\": {\"trackNumber\":\"1234\"},\"originDate\": \"2019-03-17\",\"departure\": {\"scheduled\": \"2017-04-05T12:27:00-04:00\",\"estimated\":\"2017-04-05T12:27:00-04:00\",\"terminal\": \"N\",\"gate\": \"D47\"}}"]}'
```

:smiley:
