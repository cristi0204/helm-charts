# create orc8r admin user
k -n orc8r exec deploy/orc8r-orchestrator -- /var/opt/magma/bin/accessc add-existing -admin -cert /var/opt/magma/certs/admin-operator/tls.crt admin_operator
# verify orc8r users
k -n orc8r exec deploy/orc8r-orchestrator -- /var/opt/magma/bin/accessc list-certs

# create host.nms admin user
k -n orc8r exec -it deploy/nms-magmalte -- yarn setAdminPassword host cristian-marian.radu@atos.net P@rola123

https://github.com/ShubhamTatvamasi/magma-charts-04-09-2023/blob/master/charts/orc8r/charts/certs/templates/NOTES.txt
# export  root certificates
k -n orc8r get secrets orc8r-root-tls -o jsonpath='{.data.tls\.crt}' | base64 -d > rootCA.pem

# export controller certificates
k -n orc8r get secrets orc8r-controller-tls -o jsonpath='{.data.tls\.crt}' | base64 -d > controller.pem
k -n orc8r get secrets orc8r-controller-tls -o jsonpath='{.data.tls\.key}' | base64 -d > controller.key

# export admin-operator certs
k -n orc8r describe secrets orc8r-admin-operator-tls
k -n orc8r get secrets orc8r-admin-operator-tls -o jsonpath='{.data.keystore\.p12}' | base64 -d > admin-operator.key.p12

# get the p12 password
k -n orc8r get secrets orc8r-admin-operator-pkcs12-password -o jsonpath='{.data.password}' | base64 -d
# check admin p12 password
k -n orc8r get secrets orc8r-admin-operator-tls -o jsonpath='{.data.keystore\.p12}' | base64 -d | openssl pkcs12 -nodes -passin pass:"password" -in -
