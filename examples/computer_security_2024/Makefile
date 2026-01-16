SHELL:=/bin/bash
VENV:=../../.venv
PYTHON:=~/.pyenv/shims/python #$(VENV)/bin/python3
MAKEFLAGS+=--no-print-directory

LIMITED_TRAFFIC="--limited-traffic"
CONFIG?=./config.json
SEED?=42
PARALLEL?=1

SUBSETS_RATIOS=$$(cat ${CONFIG} | ${PYTHON} -c "import sys, json; print(' '.join(str(x) for x in json.load(sys.stdin)['target_subset_ratios']))")
PRUNING_AMOUNTS=$$(cat ${CONFIG} | ${PYTHON} -c "import sys, json; print(' '.join(str(x) for x in json.load(sys.stdin)['prune_ratios']))")
SCENARIOS=$$(cat ${CONFIG} | ${PYTHON} -c "import sys, json; print(' '.join(str(x) for x in json.load(sys.stdin)['scenarios']))")
PERF_DROPS=$$(cat ${CONFIG} | ${PYTHON} -c "import sys, json; print(' '.join(str(x) for x in json.load(sys.stdin)['performance_drops']))")
CMD="${PYTHON} main.py -c ${CONFIG} -s ${SEED} -p ${PARALLEL} -v -a "

all: dataset correlations evaluation train baselines compare_pruning recursive_subset_search oracle_to_oracle oracle_to_pruned_oracle oracle_to_oracle_subsets_features oracle_to_pruned_oracle_subsets_features oracle_to_pruned_oracle_subsets_features_prune_first

dataset:
	${PYTHON} main.py -c ${CONFIG} -s ${SEED} -p ${PARALLEL} -v -a dataset

correlations:
	${PYTHON} main.py -c ${CONFIG} -s ${SEED} -p ${PARALLEL} -v -a correlations

evaluation:
	$${CMD} train_evaluation ${LIMITED_TRAFFIC}
	$${CMD} test_evaluation ${LIMITED_TRAFFIC}
	( for r in $${PRUNING_AMOUNTS}; do ($${CMD} train_evaluation ${LIMITED_TRAFFIC} -pr $${r} && $${CMD} test_evaluation ${LIMITED_TRAFFIC} -pr $${r}); done )

train:
	$${CMD} train

baselines:
	$${CMD} baselines

compare_pruning:
	$${CMD} compare_pruning

recursive_subset_search:
	$${CMD} recursive_subset_search --fixed-rank --zero-first ${LIMITED_TRAFFIC}
	$${CMD} recursive_subset_search --fixed-rank ${LIMITED_TRAFFIC}
	$${CMD} recursive_subset_search --zero-first ${LIMITED_TRAFFIC}
	$${CMD} recursive_subset_search ${LIMITED_TRAFFIC}

oracle_to_oracle:
	$${CMD} run_scenario --scenario-name o_to_o ${LIMITED_TRAFFIC}

oracle_to_pruned_oracle:
	$${CMD} only_pruning ${LIMITED_TRAFFIC} && for d in $${PERF_DROPS}; do $${CMD} run_scenario --scenario-name o_to_po ${LIMITED_TRAFFIC} --performance-drop $${d}; done

oracle_to_oracle_subsets_features:
	for r in $${SUBSETS_RATIOS}; do $${CMD} only_stochastic_search --subset-ratio $${r} ${LIMITED_TRAFFIC} && for d in $${PERF_DROPS}; do $${CMD} run_scenario --scenario-name o_to_eo ${LIMITED_TRAFFIC}  --subset-ratio $${r} --performance-drop $${d}; done; done

oracle_to_pruned_oracle_subsets_features:
	for r in $${SUBSETS_RATIOS}; do $${CMD} stochastic_search_then_pruning --subset-ratio $${r} ${LIMITED_TRAFFIC} && for d in $${PERF_DROPS}; do $${CMD} run_scenario --scenario-name o_to_ec ${LIMITED_TRAFFIC} --subset-ratio $${r} --performance-drop $${d}; done; done

oracle_to_pruned_oracle_subsets_features_prune_first:
	for r in $${SUBSETS_RATIOS}; do $${CMD} pruning_then_stochastic_search --subset-ratio $${r} ${LIMITED_TRAFFIC} && for d in $${PERF_DROPS}; do $${CMD} run_scenario --scenario-name o_to_ec ${LIMITED_TRAFFIC} --subset-ratio $${r} --performance-drop $${d} --prune-first; done; done
