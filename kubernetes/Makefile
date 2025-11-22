# ------------------------------- Variables -------------------------------- #
release_name ?= monitoring
namespace ?= monitoring
prometheus_stack_values_file ?= ./_helm/kube-prometheus-stack/values.yaml
app_namespace ?= default
logging_namespace ?= logging
fluentbit_release_name ?= fluentbit
tracing_namespace ?= opentelemetry-operator-system	# must be observability for jaeger operator to be deployed

#### ------------------------------- Targets ---------------------------------- ####
# ------------------------------- Full Deployment ---------------------------------- #
.PHONY: deploy-full-stack
deploy-full-stack: deploy-monitoring-stack deploy-logging-stack deploy-app-base-env
	@echo "Full stack deployed."

.PHONY: delete-full-stack
delete-full-stack: delete-app-base-env delete-logging-stack delete-monitoring-stack
	@echo "Full stack destroyed."

# ------------------------------- Prometheus ---------------------------------- #
.PHONY: deploy-monitoring-stack
deploy-monitoring-stack: create-prometheus-stack-release
	@echo "Monitoring stack deployed."

.PHONY: delete-monitoring-stack
delete-monitoring-stack: delete-prometheus-stack-release delete-prometheus-stack-namespace
	@echo "Monitoring stack destroyed."

.PHONY: create-prometheus-stack-release
create-prometheus-stack-release: create-prometheus-stack-namespace add-prometheus-community-chart
	helm install $(release_name) prometheus-community/kube-prometheus-stack \
		--namespace $(namespace) \
		--values $(prometheus_stack_values_file) || true

.PHONY: delete-prometheus-stack-release
delete-prometheus-stack-release:
	helm uninstall $(release_name) --namespace $(namespace) || true

.PHONY: create-prometheus-stack-namespace
create-prometheus-stack-namespace:
	kubectl create namespace $(namespace) || true

.PHONY: delete-prometheus-stack-namespace
delete-prometheus-stack-namespace:
	kubectl delete namespace $(namespace) || true

.PHONY: add-prometheus-community-chart
add-prometheus-community-chart:
	helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
	helm repo update


.PHONY: get-granfana-admin-password
get-granfana-admin-password:
	@echo "Grafana Admin Password:"
	@kubectl --namespace $(namespace) get secrets monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo

.PHONY: port-forward-grafana
port-forward-grafana:
	@echo "Access Grafana at http://localhost:3000"
	@kubectl --namespace $(namespace) port-forward svc/$(release_name)-grafana 3000:80

.PHONY: port-forward-prometheus
port-forward-prometheus:
	@echo "Access Prometheus at http://localhost:9090"
	@kubectl --namespace $(namespace) port-forward svc/$(release_name)-kube-prometheus-prometheus 9090:9090

.PHONY: port-forward-service-a
port-forward-service-a:
	@echo "Access Service A at http://localhost:8080"
	@kubectl --namespace $(app_namespace) port-forward svc/service-a 8080:80

.PHONY: port-forward-service-b
port-forward-service-b:
	@echo "Access Service B at http://localhost:8081"
	@kubectl --namespace $(app_namespace) port-forward svc/service-b 8081:80

# ------------------------------- Environments ---------------------------------- #
.PHONY: deploy-app-base-env
deploy-app-base-env: create-prometheus-stack-release
	@echo "Deploying base environment..."
	@kubectl apply --kustomize ./_base

.PHONY: delete-app-base-env
delete-app-base-env:
	@echo "Destroying base environment..."
	@kubectl delete --kustomize ./_base || true
	@$(MAKE) delete-prometheus-stack-release

# ------------------------------- EFK ---------------------------------- #
.PHONY: deploy-logging-stack
deploy-logging-stack:
	@echo "Deploying logging stack..."
	@$(MAKE) deploy-eck
	@echo "Sleeping for 1/2 minute"
	@sleep 30
	@echo "Deploying Fluent Bit..."
	@$(MAKE) deploy-fluentbit

.PHONY: delete-logging-stack
delete-logging-stack: delete-fluentbit delete-eck delete-logging-namespace
	@echo "Deleted logging stack..."

.PHONY: deploy-eck # elastic cloud on kubernetes
deploy-eck: create-logging-namespace
	@echo "Deploying ECK environment..."
	@kubectl apply -f https://download.elastic.co/downloads/eck/3.2.0/crds.yaml
	@kubectl apply -f https://download.elastic.co/downloads/eck/3.2.0/operator.yaml
	@kubectl apply --namespace $(logging_namespace) --filename ./_logging

.PHONY: delete-eck
delete-eck:
	@echo "Deleting ECK environment..."
	@kubectl delete --namespace $(logging_namespace) --filename ./_logging || true
	@kubectl delete -f https://download.elastic.co/downloads/eck/3.2.0/operator.yaml || true
	@kubectl delete -f https://download.elastic.co/downloads/eck/3.2.0/crds.yaml || true

.PHONY: port-forward-kibana
port-forward-kibana:
	@echo "Access Kibana at https://localhost:5601"
	@kubectl --namespace $(logging_namespace) port-forward svc/kibana-kb-http 5601:5601

.PHONY: get-kibana-user-creds
get-kibana-user-creds:
	@echo "Kibana User: elastic"
	@echo "Kibana Password:"
	@kubectl get secret elasticsearch-es-elastic-user -n $(logging_namespace) -o go-template='{{.data.elastic | base64decode}}' ; echo

.PHONY: create-logging-namespace
create-logging-namespace:
	kubectl create namespace $(logging_namespace) || true

.PHONY: delete-logging-namespace
delete-logging-namespace:
	kubectl delete namespace $(logging_namespace) || true

.PHONY: add-fluentbit-chart
add-fluentbit-chart:
	helm repo add fluent https://fluent.github.io/helm-charts
	helm repo update

.PHONY: deploy-fluentbit
deploy-fluentbit: add-fluentbit-chart
	@echo "Deploying Fluent Bit..."
	@helm install $(fluentbit_release_name) fluent/fluent-bit \
		--namespace $(logging_namespace) \
		--values ./_helm/fluentbit/values.yaml || true

.PHONY: delete-fluentbit
delete-fluentbit:
	@echo "Deleting Fluent Bit..."
	@helm uninstall $(fluentbit_release_name) --namespace $(logging_namespace) || true

# ------------------------------- Cluster ---------------------------------- #
.PHONY: create-cluster
create-cluster:
	@echo "Creating cluster with minikube..."
	@minikube start --nodes 2

.PHONY: destroy-cluster
destroy-cluster:
	@echo "Destorying cluster"
	@minikube delete

# ------------------------------- Jaeger ---------------------------------- #