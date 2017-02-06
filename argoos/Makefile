.PHONY: deploy deploy-with-token deploy-secured clean

NS=kube-system
TOKENSIZE:=32
TOKEN:=$(shell < /dev/urandom tr -dc '[:alnum:]' | head -c$(TOKENSIZE)) 

# Deploy Argoos basic
deploy:
	kubectl create -f deploy

# Deploy a token protected Argoos server
deploy-with-token: _gen-token
	kubectl create -f deploy-with-token/

# Deploy Argoos listening SSL
deploy-secured:
	kubectl create -f deploy-secured/

# Print token to use with Docker Registry
print-token:
	@kubectl get secret argoos-token --namespace=$(NS) -o yaml | awk '/token: /{print $$2}' | base64 -d

# Generate SSL key pair
gen-ssl: _ssl
	kubectl create secret tls argoos-certificates --namespace=$(NS) --key=server.key --cert=server.crt

# Remove Argoos from kubernetes
clean:
	kubectl delete deployment argoos --namespace=$(NS)
	kubectl delete svc        argoos --namespace=$(NS)


## Private directives

_gen-token:
	@kubectl get secret argoos-token --namespace=$(NS) 2>&1 1>/dev/null ||     \
	 kubectl create secret generic argoos-token                                \
		--from-literal=token=$(TOKEN)                                          \
		--namespace=$(NS) 2>/dev/null
	@sleep 0.5
	@echo -e "Token to set to X-Argoos-Token in docker registry configuration: \n$(shell $(MAKE) print-token)"

	
_del-token:
	kubectl delete secret argoos-token --namespace=$(NS) || :

_ssl: server.key server.crt

server.key:
	openssl genrsa -out server.key 2048

server.crt:
	openssl req -new -x509 -sha256 -key server.key -out server.crt -days 3650
	
