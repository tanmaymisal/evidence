(.venv) solution student$ python -m claims_intake.run --all
run_dir: /workspace/Build a Claims Intake Agent with a stop_reason-Driven Loop/exercises/03-dynamic-decomposition/solution/runs/20260924_124320
[claim_01_kitchen_fire] policy=POL-1001 expected=routed
  → outcome=incomplete turns=2 clarifications=0 tokens=6375/451 est=$0.0086
[claim_02_stolen_bike] policy=POL-1007 expected=routed
  → outcome=incomplete turns=3 clarifications=0 tokens=9129/430 est=$0.0113
[claim_03_water_damage] policy=POL-1004 expected=routed
  → outcome=incomplete turns=3 clarifications=0 tokens=9090/426 est=$0.0112
[claim_04_neighbor_injury] policy=POL-1002 expected=routed
  → outcome=routed turns=5 clarifications=0 tokens=17719/919 est=$0.0223
[claim_05_auto_collision] policy=POL-1003 expected=routed
  → outcome=routed turns=4 clarifications=0 tokens=14947/952 est=$0.0197
[claim_06_low_confidence_escalation] policy=POL-1008 expected=escalated
  → outcome=incomplete turns=3 clarifications=0 tokens=9454/601 est=$0.0125
[claim_07_tree_falls_on_car] policy=POL-1005 expected=routed
  → outcome=incomplete turns=2 clarifications=0 tokens=6072/329 est=$0.0077
[claim_08_minor_porch_damage] policy=POL-1006 expected=routed
  → outcome=incomplete turns=3 clarifications=0 tokens=10314/796 est=$0.0143
summary: /workspace/Build a Claims Intake Agent with a stop_reason-Driven Loop/exercises/03-dynamic-decomposition/solution/runs/20260924_124320/summary.md
total estimated cost: $0.1076
