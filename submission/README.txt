  AI Assignment 2026


Contents
--------
reasoning_agent.ipynb   Complete implementation (parsers, three classical reasoning engines,
                        engine selection), evaluation on RuleTaker / FOLIO / CounterBench with
                        final accuracy and inference time printed, experiments, reasoning traces,
                        and generation of submission.csv.
report.pdf              Representation, reasoning algorithms, experiments, reasoning traces.
submission.csv          Predictions (columns: id, flag) - see "Running on the private test set".
README.txt              This file.

Requirements
------------
Python 3.9+ with: pandas, pyarrow (for the RuleTaker parquet files), huggingface_hub, jupyter.
    pip install pandas pyarrow huggingface_hub jupyter
No GPU, no machine-learning model and no generative AI is used. Everything else is the Python
standard library (re, itertools, heapq, collections).

How to run
----------
Open reasoning_agent.ipynb and run all cells (Kernel > Restart & Run All), or:
    jupyter nbconvert --to notebook --execute --inplace reasoning_agent.ipynb
The public datasets are downloaded automatically from Hugging Face into ./data
(internet access needed the first time). A full run takes about 10 minutes on a laptop
(most of it is the 151,911-query RuleTaker test split and the optional experiments;
set RUN_EXPERIMENTS = False in section 0 to skip the experiments).

Running on the private test set
-------------------------------
In section 0 set
    TEST_PATH = '<path to the test file>'      (CSV, JSON, JSONL, TSV or Parquet)
The loader looks for an id column (id / example_id / ...), a knowledge-base column
(context / premises / premises-FOL / theory / given_info / ...) and a query column
(statement / conclusion / conclusion-FOL / question / query / ...). Multi-premise contexts may be
newline-separated, several formulas on one line (separated by '.', ';' or ','), or a Python list.
If the file also contains a label column (flag / label / answer) the accuracy is printed too.
The predictions are written to submission.csv with columns  id, flag  (True / False).
The name of the prediction column can be changed with LABEL_COLUMN in section 0.
If no test file is found, submission.csv contains the predictions for the public evaluation
sets (RuleTaker test split, FOLIO train+validation, CounterBench), which is the file included here.

Accepted input formats
----------------------
* First-order logic, Unicode:  ∀x (Young(x) → Smart(x))   ∃  ∧  ∨  ¬  ⊕  ↔  =  ≠
* First-order logic, ASCII:    forall x. (Young(x) -> Smart(x))   &  |  ~  <->  !=
* Word operators:              forall x (Young(x) implies Smart(x))  and  or  not
* Prolog clauses:              smart(X) :- young(X), \+ lazy(X).
* Templated English (RuleTaker style): "If someone is young then they are smart."
* Causal stories / counterfactual questions (CounterBench style), in English or as
  propositional logic:  (Ziklo ∧ ¬Blaf) → Trune ,  Blaf ~ Bern(0.6) ,  observed literals.

Reasoning engines (all classical)
---------------------------------
A. Forward chaining with unification and stratified negation-as-failure (closed world) -
   used for rule bases of literals with unary/binary predicates (RuleTaker semantics).
B. Resolution refutation with set-of-support, best-first (uniform-cost) clause selection,
   unification, factoring and subsumption (open world) - general first-order logic (FOLIO).
C. Structural causal model: forward propagation in topological order plus abduction by
   exhaustive search over the exogenous variables (counterfactuals, CounterBench).

Public results (from the executed notebook)
-------------------------------------------
See the summary table printed in section 11 of the notebook and in report.pdf.
