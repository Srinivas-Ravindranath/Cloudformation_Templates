# Variables
STACK_NAME := cloud-db-demo
TEMPLATE_FILE := postgres_deployment.yaml
REGION := us-east-1
KEY_PAIR_PATH := vockey.pem
SSH_USER := ec2-user
RDS_USER := ecs
RDS_PWd := Qwerty12345-
DB_NAME := wordpressdb

# Targets
.PHONY: build-stack get-outputs deploy sshec2 sshrds cleanup

# Create or update the CloudFormation stack
build-stack:
	aws cloudformation deploy \
		--template-file $(TEMPLATE_FILE) \
		--stack-name $(STACK_NAME) \
		--region $(REGION) \
		--capabilities CAPABILITY_NAMED_IAM

# Retrieve the EC2 IP and RDS host from CloudFormation outputs
get-outputs:
	$(eval EC2_IP := $(shell aws cloudformation describe-stacks \
		--stack-name $(STACK_NAME) \
		--region $(REGION) \
		--query "Stacks[0].Outputs[?OutputKey=='EC2IP'].OutputValue" \
		--output text))
	$(eval RDS_HOST := $(shell aws cloudformation describe-stacks \
		--stack-name $(STACK_NAME) \
		--region $(REGION) \
		--query "Stacks[0].Outputs[?OutputKey=='RDSHost'].OutputValue" \
		--output text))
	@echo "EC2 IP: $(EC2_IP)"
	@echo "RDS Host: $(RDS_HOST)"
	@echo "export EC2_IP=$(EC2_IP)" > stack_outputs.sh
	@echo "export RDS_HOST=$(RDS_HOST)" >> stack_outputs.sh
	@echo "Outputs saved to stack_outputs.sh"

# Build stack and get outputs in one command
deploy: build-stack get-outputs

sshec2:
	@if [ -z "$(EC2_IP)" ]; then \
	  source stack_outputs.sh; \
	fi; \
	echo "EC2 IP: $$EC2_IP" && \
	ssh -i $(KEY_PAIR_PATH) $(SSH_USER)@$$EC2_IP


sshrds:
	@if [ -z "$(RDS_HOST)" ]; then \
	  source stack_outputs.sh; \
	fi; \
	export PGPASSWORD=${RDS_PWd} && \
	psql --host=$$RDS_HOST --username=$(RDS_USER) --dbname=$(DB_NAME)

cleanup:
	@echo "Deleting CloudFormation stack: $(STACK_NAME)"
	aws cloudformation delete-stack \
		--stack-name $(STACK_NAME) \
		--region $(REGION)
	@echo "Waiting for stack deletion to complete..."
	aws cloudformation wait stack-delete-complete \
		--stack-name $(STACK_NAME) \
		--region $(REGION)
	@echo "Stack $(STACK_NAME) deleted successfully."