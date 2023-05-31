# create orc8r admin user
k -n orc8r exec deploy/orc8r-orchestrator -- /var/opt/magma/bin/accessc add-existing -admin -cert /var/opt/magma/certs/admin-operator/tls.crt admin_operator
# verify orc8r users
k -n orc8r exec deploy/orc8r-orchestrator -- /var/opt/magma/bin/accessc list-certs

# create host.nms admin user
k -n orc8r exec -it deploy/nms-magmalte -- yarn setAdminPassword host cristian-marian.radu@atos.net P@rola123
