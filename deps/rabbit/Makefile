PROJECT = rabbit
PROJECT_DESCRIPTION = RabbitMQ
PROJECT_MOD = rabbit
PROJECT_REGISTERED = rabbit_amqqueue_sup \
		     rabbit_direct_client_sup \
		     rabbit_log \
		     rabbit_node_monitor \
		     rabbit_router

define PROJECT_ENV
[
	    {tcp_listeners, [5672]},
	    {num_tcp_acceptors, 10},
	    {ssl_listeners, []},
	    {num_ssl_acceptors, 10},
	    {ssl_options, []},
	    {vm_memory_high_watermark, 0.6},
	    {vm_memory_calculation_strategy, rss},
	    {disk_free_limit, 50000000}, %% 50MB
	    {backing_queue_module, rabbit_variable_queue},
	    %% 0 ("no limit") would make a better default, but that
	    %% breaks the QPid Java client
	    {frame_max, 131072},
	    %% see rabbitmq-server#1593
	    {channel_max, 2047},
	    {ranch_connection_max, infinity},
	    {heartbeat, 60},
	    {msg_store_file_size_limit, 16777216},
	    {msg_store_shutdown_timeout, 600000},
	    {fhc_write_buffering, true},
	    {fhc_read_buffering, false},
	    {queue_index_max_journal_entries, 32768},
	    {queue_index_embed_msgs_below, 4096},
	    {default_user, <<"guest">>},
	    {default_pass, <<"guest">>},
	    {default_user_tags, [administrator]},
	    {default_vhost, <<"/">>},
	    {default_permissions, [<<".*">>, <<".*">>, <<".*">>]},
	    {loopback_users, [<<"guest">>]},
	    {password_hashing_module, rabbit_password_hashing_sha256},
	    {server_properties, []},
	    {collect_statistics, none},
	    {collect_statistics_interval, 5000},
	    {mnesia_table_loading_retry_timeout, 30000},
	    {mnesia_table_loading_retry_limit, 10},
	    %% The identity to act as for anonymous logins.
	    {anonymous_login_user, <<"guest">>},
	    {anonymous_login_pass, <<"guest">>},
	    %% "The server mechanisms are ordered in decreasing level of preference."
	    %% AMQP §5.3.3.1
	    {auth_mechanisms, ['PLAIN', 'AMQPLAIN', 'ANONYMOUS']},
	    {auth_backends, [rabbit_auth_backend_internal]},
	    {delegate_count, 16},
	    {trace_vhosts, []},
	    {ssl_cert_login_from, distinguished_name},
	    {ssl_handshake_timeout, 5000},
	    {ssl_allow_poodle_attack, false},
	    {handshake_timeout, 10000},
	    {reverse_dns_lookups, false},
	    {cluster_partition_handling, ignore},
	    {cluster_keepalive_interval, 10000},
	    {autoheal_state_transition_timeout, 60000},
	    {tcp_listen_options, [{backlog,       128},
	                          {nodelay,       true},
	                          {linger,        {true, 0}},
	                          {exit_on_close, false}
	                         ]},
	    {ssl_apps, [asn1, crypto, public_key, ssl]},
	    %% see rabbitmq-server#227 and related tickets.
	    %% msg_store_credit_disc_bound only takes effect when
	    %% messages are persisted to the message store. If messages
	    %% are embedded on the queue index, then modifying this
	    %% setting has no effect because credit_flow is not used when
	    %% writing to the queue index. See the setting
	    %% queue_index_embed_msgs_below above.
	    {msg_store_credit_disc_bound, {4000, 800}},
	    {msg_store_io_batch_size, 4096},
	    %% see rabbitmq-server#143,
	    %% rabbitmq-server#949, rabbitmq-server#1098
	    {credit_flow_default_credit, {400, 200}},
	    {quorum_commands_soft_limit, 32},
	    {quorum_cluster_size, 3},
	    %% see rabbitmq-server#248
	    %% and rabbitmq-server#667
	    {channel_operation_timeout, 15000},
	    %% See https://www.rabbitmq.com/consumers.html#acknowledgement-timeout
	    %% 30 minutes
	    {consumer_timeout, 1800000},

	    %% used by rabbit_peer_discovery_classic_config
	    {cluster_nodes, {[], disc}},

	    {config_entry_decoder, [{passphrase, undefined}]},
	    {background_gc_enabled, false},
	    {background_gc_target_interval, 60000},
	    %% rabbitmq-server#589
	    {proxy_protocol, false},
	    {disk_monitor_failure_retries, 10},
	    {disk_monitor_failure_retry_interval, 120000},
	    %% either "stop_node" or "continue".
	    %% by default we choose to not terminate the entire node if one
	    %% vhost had to shut down, see server#1158 and server#1280
	    {vhost_restart_strategy, continue},
	    %% {global, prefetch count}
	    {default_consumer_prefetch, {false, 0}},
		  %% interval at which the channel can perform periodic actions
	    {channel_tick_interval, 60000},
	    %% Default max message size is 16 MB
	    {max_message_size, 16777216},
	    %% Socket writer will run GC every 1 GB of outgoing data
	    {writer_gc_threshold, 1000000000},
	    %% interval at which connection/channel tracking executes post operations
	    {tracking_execution_timeout, 15000},
	    {stream_messages_soft_limit, 256},
      	{track_auth_attempt_source, false},
		{credentials_obfuscation_fallback_secret, <<"nocookie">>},
      	{dead_letter_worker_consumer_prefetch, 32},
      	{dead_letter_worker_publisher_confirm_timeout, 180000},
		%% EOL date for the current release series, if known/announced
		{release_series_eol_date, none},
		{vhost_process_reconciliation_run_interval, 30},
		%% for testing
		{vhost_process_reconciliation_enabled, true}
	  ]
endef

LOCAL_DEPS = sasl os_mon inets compiler public_key crypto ssl syntax_tools xmerl

BUILD_DEPS = rabbitmq_cli
DEPS = ranch rabbit_common amqp10_common rabbitmq_prelaunch ra sysmon_handler stdout_formatter recon redbug observer_cli osiris syslog systemd seshat khepri khepri_mnesia_migration cuttlefish gen_batch_server
TEST_DEPS = rabbitmq_ct_helpers rabbitmq_ct_client_helpers meck proper amqp_client rabbitmq_amqp_client rabbitmq_amqp1_0

PLT_APPS += mnesia runtime_tools

dep_syslog = git https://github.com/schlagert/syslog 4.0.0
dep_osiris = git https://github.com/rabbitmq/osiris v1.8.3
dep_systemd = hex 0.6.1

define usage_xml_to_erl
$(subst __,_,$(patsubst $(DOCS_DIR)/rabbitmq%.1.xml, src/rabbit_%_usage.erl, $(subst -,_,$(1))))
endef

DOCS_DIR     = docs
MANPAGES     = $(wildcard $(DOCS_DIR)/*.[0-9])
WEB_MANPAGES = $(patsubst %,%.html,$(MANPAGES))
MD_MANPAGES  = $(patsubst %,%.md,$(MANPAGES))

DEP_EARLY_PLUGINS = rabbit_common/mk/rabbitmq-early-plugin.mk
DEP_PLUGINS = rabbit_common/mk/rabbitmq-plugin.mk

include ../../rabbitmq-components.mk
include ../../erlang.mk

ifeq ($(strip $(BATS)),)
BATS := $(ERLANG_MK_TMP)/bats/bin/bats
endif

BATS_GIT ?= https://github.com/sstephenson/bats
BATS_COMMIT ?= v0.4.0

$(BATS):
	$(verbose) mkdir -p $(ERLANG_MK_TMP)
	$(gen_verbose) git clone --depth 1 --branch=$(BATS_COMMIT) $(BATS_GIT) $(ERLANG_MK_TMP)/bats

.PHONY: bats

bats: $(BATS)
	$(verbose) $(BATS) $(TEST_DIR)

tests:: bats

SLOW_CT_SUITES := backing_queue \
		  channel_interceptor \
		  cluster \
		  cluster_rename \
		  clustering_management \
		  config_schema \
		  confirms_rejects \
		  consumer_timeout \
		  crashing_queues \
		  dynamic_ha \
		  dynamic_qq \
		  eager_sync \
		  feature_flags \
		  health_check \
		  many_node_ha \
		  metrics \
		  msg_store \
		  partitions \
		  per_user_connection_tracking \
		  per_vhost_connection_limit \
		  per_vhost_connection_limit_partitions \
		  per_vhost_msg_store \
		  per_vhost_queue_limit \
		  policy \
		  priority_queue \
		  priority_queue_recovery \
		  publisher_confirms_parallel \
		  queue_parallel \
		  quorum_queue \
		  rabbit_core_metrics_gc \
		  rabbit_fifo_prop \
		  rabbitmq_queues_cli_integration \
		  rabbitmqctl_integration \
		  simple_ha \
		  sync_detection \
		  unit_inbroker_non_parallel \
		  unit_inbroker_parallel \
		  vhost
FAST_CT_SUITES := $(filter-out $(sort $(SLOW_CT_SUITES)),$(CT_SUITES))

ct-fast:
	$(MAKE) ct CT_SUITES='$(FAST_CT_SUITES)'

ct-slow:
	$(MAKE) ct CT_SUITES='$(SLOW_CT_SUITES)'

CT_OPTS += -ct_hooks rabbit_ct_hook []

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
	ct_master:run("ct.test.spec"),
	peer:stop(Pid4),
	peer:stop(Pid3),
	peer:stop(Pid2),
	peer:stop(Pid1),
	halt()
endef

ct-master: test-build
	$(verbose) mkdir -p $(CT_LOGS_DIR)
	$(call erlang,$(ct_master.erl),-sname rabbit_master@localhost -hidden -kernel net_ticktime 5)

# --------------------------------------------------------------------
# Compilation.
# --------------------------------------------------------------------

RMQ_ERLC_OPTS += -I $(DEPS_DIR)/rabbit_common/include
EDOC_OPTS += {preprocess,true},{includes,["."]}

ifdef INSTRUMENT_FOR_QC
RMQ_ERLC_OPTS += -DINSTR_MOD=gm_qc
EDOC_OPTS += ,{macros,[{'INSTR_MOD',gm_qc}]}
else
RMQ_ERLC_OPTS += -DINSTR_MOD=gm
EDOC_OPTS += ,{macros,[{'INSTR_MOD',gm}]}
endif

ifdef CREDIT_FLOW_TRACING
RMQ_ERLC_OPTS += -DCREDIT_FLOW_TRACING=true
endif

ifdef TRACE_SUPERVISOR2
RMQ_ERLC_OPTS += -DTRACE_SUPERVISOR2=true
endif

# --------------------------------------------------------------------
# Documentation.
# --------------------------------------------------------------------

.PHONY: manpages web-manpages distclean-manpages

docs:: manpages web-manpages

manpages: $(MANPAGES)
	@:

web-manpages: $(WEB_MANPAGES) $(MD_MANPAGES)
	@:

# We use mandoc(1) to convert manpages to HTML plus an awk script which
# does:
#     1. remove tables at the top and the bottom (they recall the
#        manpage name, section and date)
#     2. "downgrade" headers by one level (eg. h1 -> h2)
#     3. annotate .Dl lines with more CSS classes
%.html: %
	$(gen_verbose) mandoc -T html -O 'fragment,man=%N.%S.html' "$<" | \
	  awk '\
	  /^<table class="head">$$/ { remove_table=1; next; } \
	  /^<table class="foot">$$/ { remove_table=1; next; } \
	  /^<\/table>$$/ { if (remove_table) { remove_table=0; next; } } \
	  { if (!remove_table) { \
	    line=$$0; \
	    gsub(/<h2/, "<h3", line); \
	    gsub(/<\/h2>/, "</h3>", line); \
	    gsub(/<h1/, "<h2", line); \
	    gsub(/<\/h1>/, "</h2>", line); \
	    gsub(/class="D1"/, "class=\"D1 lang-bash\"", line); \
	    gsub(/class="Bd Bd-indent"/, "class=\"Bd Bd-indent lang-bash\"", line); \
	    gsub(/&#[xX]201[cCdD];/, "\\&quot;", line); \
	    print line; \
	  } } \
	  ' > "$@"

%.md: %
	$(gen_verbose) mandoc -T markdown -O 'fragment,man=%N.%S.md' "$<" | \
	  sed -E -e 's/\{/\&lcub;/g' \
	> "$@"

distclean:: distclean-manpages

distclean-manpages::
	$(gen_verbose) rm -f $(WEB_MANPAGES) $(MD_MANPAGES)
