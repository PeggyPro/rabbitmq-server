PROJECT = rabbitmq_peer_discovery_consul
PROJECT_DESCRIPTION = Consult-based RabbitMQ peer discovery backend

DEPS = rabbit_common rabbitmq_peer_discovery_common rabbit
TEST_DEPS = rabbitmq_ct_helpers rabbitmq_ct_client_helpers ct_helper meck
dep_ct_helper = git https://github.com/extend/ct_helper.git master

DEP_EARLY_PLUGINS = rabbit_common/mk/rabbitmq-early-plugin.mk
DEP_PLUGINS = rabbit_common/mk/rabbitmq-plugin.mk

include ../../rabbitmq-components.mk
include ../../erlang.mk
