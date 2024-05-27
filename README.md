# Robust evaluation of political worldview in LLMs

## Coding information for the IDs 

String with the following format:

    CountryCode + _ + StatementID + _ + Original|Question|ParaphraseID|Opposite|Negation|Country-Agnostic|Translation + _ + LanguageCode

e.g. _ch_15_0000001_fr_

* Original, opposite, question, negation, country-agnostic and translation are binary.
* **StatementID** is the ID of the statement in the original dataset.
* **ParaphraseID** = 0 means it's not a paraphrase. From 1 on it's the ID.
* **country-agnostic**: 0 if specific else 1 if agnostic
* **Original** 1 if original statement else 0
* **Opposite** 1 if opposite statement else 0
* **Question** 1 if question else 0 (not included in our dataset anymore)
* **Negation** 1 if negation else 0
* **Translation** 1 if translation else 0
* **Language**: e.g. en, es, de...
* **CountryCode**: e.g. ch, de, fr, it


## Chapel Hill Survey

The data from the Chapel Hill Expert Survey can be found [here](https://www.chesdata.eu/ches-europe).


# Code 

## Run models to generate answers given prompts

The code to run the models can be found in the `run_model` folder. The code is based on the Hugging Face transformers library.

[...]

## Evaluation

There are two steps of the evaluation, the reliability tests and the analysis of the political worldviews.

### Reliability tests

The reliability tests are based on the inter-rater agreement and the test-retest reliability. The code can be found in the `reliability` folder.

The first step is to compute a probability for the leaning of the model for each prompt. To do this you need to have the 
answers for each prompt mapped to `binary_answer`. The file with the binary answers (e.g. you have mapped "yes" to 1 
and "no" to 0) also needs to have a unique identifier for the statement varient (`statement_id`), the prompt template
(`template_id`), and a column indicating whether the labels in the prompt have been inverted (`inverted`). On top of that
the csv file should contain a column indicating the `model_name`.

For each prompt you are expected to have run inference several times (n = sample_size) and have the answers for each run.
N should be at least 30 to have a reliable estimate of the leaning. If you want to compute a more robust significance of
the leaning you can increase the sample size.

In order to compute the leaning of the model for each prompt, you need to run the following script:

    python -m reliability.aggregate --questionnaire_with_model_responses /path/to/answers --sample_size 30 --out_dir /path/to/output_dir

The script will also create a random baseline. The aggregated results will be stored in the output directory under
the file `aggregated_results.csv`.

In order to run the evaluation, you need to run the following script:

    python -m reliability.evaluation --results_all /path/to/aggregated_results.csv --results_random /path/to/random/random_baseline.csv --results_dir /path/to/results_dir

This script will comute reliability tests for each statement for each model-template combination.

The tests are the following:
* test_sign: check for all statement variants for prompts with non-inverted label order, whether the model had a
significant leaning towards a positive or negative leaning.

* test_label_inversion: check whether for the original statement variant the model responded the same way when inverting the labels in the prompt
(e.g. do you agree or disagree vs do you disagree or agree?)

* test_semantic_equivalence: check whether the model responses are the same for the original variant of the statement
 and three different paraphrases of it (i.e. its semantic equivalences)

* test_negation: check whether the binary value for the negation statement is different from the original statement

* test_opposite: check whether the binary value for the opposite statement is different from the original statement

To run agreement for each model instead of reliability tests, you can run the following script:

    python -m reliability.agreement --results_all /path/to/aggregated_results.csv --results_dir /path/to/results_dir

A high agreement indicates that the model is more reliable in its responses.


### Analysis of the political worldviews

The code to analyze the political worldviews can be found in the `political_biases` folder. 

In order to run the analysis concerning the political orientation of the models, you need to run the following script:

    python3 leaning_analysis.py --passed_test hard_pass -- condition 0

You can choose the type of test you want to analyse between the `test_names` options found in the utils.py file. The condition is the condition of the test you want to analyse 
-- either only statements that have agreed (condition 1), disagreed (condition 0), all statements (condition 2) or similations (condition 3).

If you want to run the analysis concerning the stance in each specific policy domain, run:

    python3 domain_specific.py --passed_test hard_pass

The arguments are the same as above. If you want run with a combination of different reliability tests, you need to modify the tests in the `test_names` dictionary in the utils.py file. 

Finally, if you want to run the number of agrees and disagrees that the models answered, you can run:

    python3 plot_agree_disagree.py --passed_test hard_pass




