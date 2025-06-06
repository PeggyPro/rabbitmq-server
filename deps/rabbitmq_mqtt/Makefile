PROJECT = rabbitmq_mqtt
PROJECT_DESCRIPTION = RabbitMQ MQTT Adapter
PROJECT_MOD = rabbit_mqtt

define PROJECT_ENV
[
	    {ssl_cert_login,false},
	    {allow_anonymous, true},
	    {vhost, <<"/">>},
	    {exchange, <<"amq.topic">>},
	    {max_session_expiry_interval_seconds, 86400}, %% 1 day
	    {retained_message_store, rabbit_mqtt_retained_msg_store_dets},
	    %% only used by DETS store
	    {retained_message_store_dets_sync_interval, 2000},
	    {prefetch, 10},
	    {ssl_listeners, []},
	    {tcp_listeners, [1883]},
	    {num_tcp_acceptors, 10},
	    {num_ssl_acceptors, 10},
	    {tcp_listen_options, [{backlog,   128},
	                          {nodelay,   true},
	                          {send_timeout, 15000},
	                          {send_timeout_close, true}
	                         ]},
	    {proxy_protocol, false},
	    {sparkplug, false},
	    {mailbox_soft_limit, 200},
	    {max_packet_size_unauthenticated, 65536},
	    %% 256 MB is upper limit defined by MQTT spec
	    %% We set 16 MB as defined in deps/rabbit/Makefile max_message_size
	    {max_packet_size_authenticated, 16777216},
	    {topic_alias_maximum, 16}
	  ]
endef

define PROJECT_APP_EXTRA_KEYS
	{broker_version_requirements, []}
endef

# We do not need QUIC as dependency of emqtt.
BUILD_WITHOUT_QUIC=1
export BUILD_WITHOUT_QUIC

LOCAL_DEPS = ssl
DEPS = ranch rabbit amqp10_common
TEST_DEPS = cowlib emqtt ct_helper rabbitmq_ct_helpers rabbitmq_ct_client_helpers rabbitmq_management amqp_client rabbitmq_consistent_hash_exchange rabbitmq_amqp_client rabbitmq_stomp rabbitmq_stream rabbitmq_federation

PLT_APPS += rabbitmq_cli elixir

dep_ct_helper = git https://github.com/extend/ct_helper.git master
dep_emqtt = git https://github.com/emqx/emqtt.git 1.11.0

CT_HOOKS = rabbit_ct_hook
DEP_EARLY_PLUGINS = rabbit_common/mk/rabbitmq-early-plugin.mk
DEP_PLUGINS = rabbit_common/mk/rabbitmq-plugin.mk

include ../../rabbitmq-components.mk
include ../../erlang.mk


clean::
	if test -d test/java_SUITE_data; then cd test/java_SUITE_data && $(MAKE) clean; fi

# Parallel CT.
#
# @todo Move most of this in common files.

define ct_master.erl
	StartOpts = #{
		host => "localhost",
		connection => standard_io,
		args => ["-hidden"]
	},
	{ok, Pid1, _} = peer:start(StartOpts#{name => "rabbit_shard1"}),
	{ok, Pid2, _} = peer:start(StartOpts#{name => "rabbit_shard2"}),
	{ok, Pid3, _} = peer:start(StartOpts#{name => "rabbit_shard3"}),
	{ok, Pid4, _} = peer:start(StartOpts#{name => "rabbit_shard4"}),
	peer:call(Pid1, net_kernel, set_net_ticktime, [5]),
	peer:call(Pid2, net_kernel, set_net_ticktime, [5]),
	peer:call(Pid3, net_kernel, set_net_ticktime, [5]),
	peer:call(Pid4, net_kernel, set_net_ticktime, [5]),
	peer:call(Pid1, persistent_term, put, [rabbit_ct_tcp_port_base, 23000]),
	peer:call(Pid2, persistent_term, put, [rabbit_ct_tcp_port_base, 25000]),
	peer:call(Pid3, persistent_term, put, [rabbit_ct_tcp_port_base, 27000]),
	peer:call(Pid4, persistent_term, put, [rabbit_ct_tcp_port_base, 29000]),
	[{[_], {ok, Results}}] = ct_master_fork:run("$1"),
	peer:stop(Pid4),
	peer:stop(Pid3),
	peer:stop(Pid2),
	peer:stop(Pid1),
	lists:foldl(fun
		({_, {_, 0, {_, 0}}}, Err) -> Err + 1;
		(What, Peer) -> halt(Peer)
	end, 1, Results),
	halt(0)
endef

PARALLEL_CT_SET_1_A = auth retainer federation feature_flag
PARALLEL_CT_SET_1_B = cluster command config config_schema mc_mqtt packet_prop \
    processor protocol_interop proxy_protocol rabbit_mqtt_confirms reader util
PARALLEL_CT_SET_1_C = java v5
PARALLEL_CT_SET_1_D = mqtt_shared

PARALLEL_CT_SUITES = $(PARALLEL_CT_SET_1_A) $(PARALLEL_CT_SET_1_B) $(PARALLEL_CT_SET_1_C) $(PARALLEL_CT_SET_1_D)

ifeq ($(filter-out $(SEQUENTIAL_CT_SUITES) $(PARALLEL_CT_SUITES),$(CT_SUITES)),)
parallel-ct-sanity-check:
	$(verbose) :
else
parallel-ct-sanity-check:
	$(verbose) printf "%s\n" \
		"In order for new test suites to be run in CI, the test suites" \
		"must be added to one of the PARALLEL_CT_SET_<N>_<M> variables." \
		"" \
		"The following test suites are missing:" \
		"$(filter-out $(SEQUENTIAL_CT_SUITES) $(PARALLEL_CT_SUITES),$(CT_SUITES))"
	$(verbose) exit 1
endif

define tpl_parallel_ct_test_spec
{logdir, "$(CT_LOGS_DIR)"}.
{logdir, master, "$(CT_LOGS_DIR)"}.
{create_priv_dir, all_nodes, auto_per_run}.
{auto_compile, false}.

{node, shard1, 'rabbit_shard1@localhost'}.
{node, shard2, 'rabbit_shard2@localhost'}.
{node, shard3, 'rabbit_shard3@localhost'}.
{node, shard4, 'rabbit_shard4@localhost'}.

{define, 'Set1', [$(call comma_list,$(addsuffix _SUITE,$1))]}.
{define, 'Set2', [$(call comma_list,$(addsuffix _SUITE,$2))]}.
{define, 'Set3', [$(call comma_list,$(addsuffix _SUITE,$3))]}.
{define, 'Set4', [$(call comma_list,$(addsuffix _SUITE,$4))]}.

{suites, shard1, "test/", 'Set1'}.
{suites, shard2, "test/", 'Set2'}.
{suites, shard3, "test/", 'Set3'}.
{suites, shard4, "test/", 'Set4'}.
endef

define parallel_ct_set_target
tpl_parallel_ct_test_spec_set_$1 = $$(call tpl_parallel_ct_test_spec,$(PARALLEL_CT_SET_$(1)_A),$(PARALLEL_CT_SET_$(1)_B),$(PARALLEL_CT_SET_$(1)_C),$(PARALLEL_CT_SET_$(1)_D))

parallel-ct-set-$(1): test-build
	$(verbose) mkdir -p $(CT_LOGS_DIR)
	$(verbose) $$(call core_render,tpl_parallel_ct_test_spec_set_$(1),ct.set-$(1).spec)
	$$(eval ERL := erl -noinput -boot no_dot_erlang)
	$$(call erlang,$$(call ct_master.erl,ct.set-$(1).spec),-sname parallel_ct_$(PROJECT)@localhost -hidden -kernel net_ticktime 5)
endef

$(foreach set,1,$(eval $(call parallel_ct_set_target,$(set))))
