PROJECT = rabbitmq_amqp_client
PROJECT_DESCRIPTION = AMQP 1.0 client for RabbitMQ

DEPS = amqp10_client
TEST_DEPS = rabbitmq_ct_helpers

BUILD_DEPS = rabbit_common
DEP_EARLY_PLUGINS = rabbit_common/mk/rabbitmq-early-plugin.mk
TEST_DEPS = rabbit rabbitmq_ct_helpers

# We do not depend on rabbit therefore can't run the broker.
DEP_PLUGINS = $(PROJECT)/mk/rabbitmq-build.mk

.DEFAULT_GOAL = all

include rabbitmq-components.mk
include erlang.mk
