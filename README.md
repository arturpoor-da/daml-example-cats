This example can be used to reproduce the 'lost transactions' issue in Daml triggers.

Run the following steps, each in a separate terminal window: 

1. ~~~~
   canton --config simple-topology.conf --auto-connect-local -v --log-file-name logs/canton.log
   ~~~~


1. ~~~~
   daml ledger upload-dar .daml/dist/cats-0.1.0.dar

   daml script --dar .daml/dist/cats-0.1.0.dar --script-name Scripts:allocate --output-file party.json --ledger-port 6865 --ledger-host localhost

   daml script --dar .daml/dist/cats-0.1.0.dar --script-name Scripts:createFoodContracts --input-file party.json --ledger-port 6865 --ledger-host localhost
   ~~~~

1. ~~~~
   export _JAVA_OPTIONS="-Dlogback.configurationFile=trigger-service-logback.xml"
   daml trigger-service --ledger-host localhost --ledger-port 6865 --dar .daml/dist/cats-0.1.0.dar | tee logs/trigger-service.log
   ~~~~

Run the following steps at the same time:

1. ~~~~
   packageId=$(daml damlc inspect .daml/dist/cats-0.1.0.dar | head -n 1 | cut -d ' ' -f 2)
   party=$(cat party.json)

   curl -H "Content-Type: application/json" --data "{\"triggerName\": \"$packageId:Trigger:trigger\", \"party\": $party, \"applicationId\": \"foo\"}" -v localhost:8088/v1/triggers
   ~~~~
   
1. ~~~~
   daml script --dar .daml/dist/cats-0.1.0.dar --script-name Scripts:archiveFoodContracts --input-file party.json --ledger-port 6865 --ledger-host localhost
   ~~~~
