# Hyperledger_External_Chaincode
External chaincode Lab



cd /external_sacc
tar cfz code.tar.gz connection.json
tar cfz sacc_external.tgz metadata.json code.tar.gz

peer lifecycle chaincode install sacc_external.tgz 

peer lifecycle chaincode approveformyorg --tls --cafile $ORDERER_CA -o localhost:8050 --channelID channel1 --name sacc --version 1 --init-required --sequence 1 --waitForEvent --package-id xxxxx
peer lifecycle chaincode approveformyorg --tls --cafile $ORDERER_CA -o localhost:7050 --channelID channel1 --name sacc --version 1 --init-required --sequence 1 --waitForEvent --package-id xxxxx

GO111MODULE=on go mod vendor
docker build -t hyperledger/sacc_ext .
docker-compose up -d sacc-ext

peer lifecycle chaincode commit --tls --cafile $ORDERER_CA -o localhost:7050 --peerAddresses $CORE_PEER_ADDRESS --tlsRootCertFiles $CORE_PEER_TLS_ROOTCERT_FILE --peerAddresses localhost:8051 --tlsRootCertFiles /tmp/hyperledger/org2/peer1/tls-msp/tlscacerts/tls-0-0-0-0-7052.pem --channelID channel1 --name sacc --version 1 --sequence 1 --init-required

peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer1-org1 --tls true --cafile $ORDERER_CA --peerAddresses localhost:7051 --tlsRootCertFiles /tmp/hyperledger/org1/peer1/assets/tls-ca/tls-ca-cert.pem --peerAddresses localhost:8051 --tlsRootCertFiles /tmp/hyperledger/org2/peer1/assets/tls-ca/tls-ca-cert.pem --channelID channel1 --name sacc --isInit -c '{"Args":["name","arnaud"]}'

peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer1-org1 --tls true --cafile $ORDERER_CA --peerAddresses localhost:7051 --tlsRootCertFiles /tmp/hyperledger/org1/peer1/assets/tls-ca/tls-ca-cert.pem --peerAddresses localhost:8051 --tlsRootCertFiles /tmp/hyperledger/org2/peer1/assets/tls-ca/tls-ca-cert.pem --channelID channel1 --name sacc -c '{"Args":["set","name","ArnaudBart"]}'

peer chaincode query -C channel1 -n sacc -c '{"Args":["get","name"]}'