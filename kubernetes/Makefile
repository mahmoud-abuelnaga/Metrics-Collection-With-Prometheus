# ------------------------------- Variables -------------------------------- #
release_name ?= monitoring
namespace ?= monitoring
prometheus_stack_values_file ?= ./_helm/kube-prometheus-stack/values.yaml

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


.PHONY: deploy-base
deploy-base: create-prometheus-stack-release
	kubectl apply --kustomize ./_base