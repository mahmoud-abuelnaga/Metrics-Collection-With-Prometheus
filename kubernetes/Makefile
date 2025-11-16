# ------------------------------- Variables -------------------------------- #
release_name ?= monitoring
namespace ?= monitoring
prometheus_stack_values_file ?= ./_helm/kube-prometheus-stack/values.yaml
app_namespace ?= default

# ------------------------------- Targets ---------------------------------- #
.PHONY: create-prometheus-stack-release
create-prometheus-stack-release: create-namespace add-prometheus-community-chart
	helm install $(release_name) prometheus-community/kube-prometheus-stack \
		--namespace $(namespace) \
		--values $(prometheus_stack_values_file) || true

.PHONY: delete-prometheus-stack-release
delete-prometheus-stack-release:
	helm uninstall $(release_name) --namespace $(namespace) || true

.PHONY: create-namespace
create-namespace:
	kubectl create namespace $(namespace) || true

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


.PHONY: deploy-base
deploy-base: create-prometheus-stack-release
	@echo "Deploying base environment..."
	@kubectl apply --kustomize ./_base


.PHONY: destroy-base
destroy-base: delete-prometheus-stack-release
	@echo "Destroying base environment..."
	@kubectl delete --kustomize ./_base || true
