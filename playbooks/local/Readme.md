
dev_admin_token=`cat /home/${USER}/apps/loki/data/dev_admin.token`

current_date=`date +%s%N` && curl -u dev:$dev_admin_token \
-H "Content-Type: application/json" \
-H "X-Scope-OrdID: dev" \
https://loki.aphm.duckdns.org/loki/api/v1/push \
--data "{\"streams\": [{ \"stream\": { \"job\": \"test\" }, \"values\": [ [ \"$current_date\", \"A log line on dev\" ] ] }]}"


alias demostop="docker stop grafana-agent keycloak mariadb minio postgres tempo grafana-enterprise loki mimir  prometheus smoke-test-app"

alias demostart="docker start grafana-agent keycloak mariadb minio postgres tempo grafana-enterprise loki mimir  prometheus smoke-test-app"




registry reposilite regui  traefik portainer