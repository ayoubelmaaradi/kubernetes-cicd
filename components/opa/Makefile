#❌⚠️✅
# COLORS
GREEN  := $(shell tput -Txterm setaf 2)
YELLOW := $(shell tput -Txterm setaf 3)
WHITE  := $(shell tput -Txterm setaf 7)
RED		 := $(shell tput -Txterm setaf 1)
CYAN	 := $(shell tput -Txterm setaf 6)
RESET  := $(shell tput -Txterm sgr0)

TOPLVL = ../..

-include $(TOPLVL)/common.mk

## fetch depdencies
fetch.chart:
	helm fetch stable/opa --untar --untardir charts

## generate ambassador resources 
generate: fetch.chart
	kubectl create namespace opa --dry-run -o yaml | kubectl apply -f -
	helm template charts/opa --set fullnameOverride=opa -f values/${CLUSTER}/values.yaml --namespace opa > resources/${CLUSTER}/opa.yaml
	helm template charts/opa --set fullnameOverride=opa -x templates/webhookconfiguration.yaml -f values/${CLUSTER}/values.yaml --namespace opa > resources/${CLUSTER}/webhookconfiguration.yaml

## install ambassador

install:
	kubectl apply -n opa -f resources/${CLUSTER}/webhookconfiguration.yaml
	kubectl apply -n opa -f resources/${CLUSTER}/opa.yaml
	kubectl label ns kube-system openpolicyagent.org/webhook=ignore
	kubectl label ns opa openpolicyagent.org/webhook=ignore


## uninstall istio from cluster
uninstall:
	kubectl delete --ignore-not-found=true -f resources/${CLUSTER}/webhookconfiguration.yaml -n opa
	kubectl delete --ignore-not-found=true -f resources/${CLUSTER}/opa.yaml -n opa
	kubectl delete namespace opa
	$(call wait_for_ns_termination,opa)


## create example configmaps
example.policy.create:
	kubectl apply -f resources/examples/greeting.cm.yaml -n opa
	kubectl apply -f resources/examples/pod-registry.cm.yaml -n opa
example.run:
	kubectl create namespace opa-example --dry-run -o yaml | kubectl apply -f -
	kubectl label namespace opa-example
###Help
## Show help
help:
	@echo ''
	@echo '######################### ISTIO MANAGER #########################'
	@echo ''
	@echo ''
	@echo 'Usage:'
	@echo ''
	@echo '  ${YELLOW}make${RESET} ${GREEN}<target>${RESET}'
	@echo ''
	@echo 'Targets:'
	@awk '/(^[a-zA-Z\-\.\_0-9]+:)|(^###[a-zA-Z]+)/ { \
		header = match($$1, /^###(.*)/); \
		if (header) { \
			title = substr($$1, 4, length($$1)); \
			printf "${CYAN}%s${RESET}\n", title; \
		} \
		helpMessage = match(lastLine, /^## (.*)/); \
		if (helpMessage) { \
			helpCommand = substr($$1, 0, index($$1, ":")-1); \
			helpMessage = substr(lastLine, RSTART + 3, RLENGTH); \
			printf "  ${YELLOW}%-$(TARGET_MAX_CHAR_NUM)s${RESET} ${GREEN}%s${RESET}\n", helpCommand, helpMessage; \
		} \
	} \
	{ lastLine = $$0 }' $(MAKEFILE_LIST)