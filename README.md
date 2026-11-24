# econ3916-lab18-model-evaluation
# Model Evaluation: The Curve, and What It Cannot Tell You

## Objective

I wanted to see what ROC and precision-recall curves can and cannot tell me
about a classifier for a rare outcome, and how the answer changes once I put
real costs on mistakes.

## Methodology

- I simulated 8,000 cases of a rare outcome and split them into training and
  test rows. 11.8% of the test rows were positive.
- I fit a logistic regression and drew its ROC curve and precision-recall
  curve on the test rows.
- I fit a boosted-tree model and compared its ROC AUC with the logistic
  regression's on the full test set.
- I ran a paired bootstrap with 500 resamples of the test rows, computing the
  AUC difference between the two models on each resample, and took the 95%
  interval of those differences.
- I built an expected-cost curve where a missed positive costs ten times as
  much as a false alarm. I chose the cheapest threshold using the training
  rows only, then priced that threshold once on the test rows. I also priced
  the default 0.5 threshold for comparison.
- I swept the cost ratio from 2:1 to 50:1 for both models, choosing each
  threshold the same way (on the training rows), and checked whether the
  cheaper model changed across the range.
- I wrote a small function, `roc_point()`, that returns the false positive
  rate and true positive rate at a single threshold, and used it at my chosen
  threshold.

## Key Findings

- **Class balance:** Only 11.8% of the test rows were positive, so
  accuracy and the ROC curve alone say little about how the model handles the
  cases that matter.
- **Curves:** The logistic regression had a ROC AUC of 0.847 and an
  average precision of 0.465, about four times the 0.118 that random
  scoring would get.
- **Model comparison:** The boosted-tree model had a ROC AUC of 0.843.
  The AUC difference on the full test set was +0.004 (logistic minus
  boosted), and the paired bootstrap gave a 95% interval of
  [−0.002, +0.009]. The interval includes zero, so I can't say one model
  ranks cases better than the other.
- **Cost-based threshold:** With a miss costing ten times a false alarm, the
  cheapest threshold on the training rows was 0.0905. On the test rows
  it cost 0.469 per case, against 0.944 at the default 0.5, roughly half the
  cost. The closed-form threshold of 0.0909 cost 0.468, essentially the same.
- **Cost ratio sweep:** Across cost ratios from 2:1 to 50:1, the logistic
  regression was cheaper at every ratio except 5:1, where the boosted model
  was cheaper. The gaps were small (at most 0.025 per case), so I don't treat
  the switch as real. This shows that a single AUC number doesn't settle which
  model to use; that depends on what mistakes cost.
- **One point on the curve:** At my chosen threshold, `roc_point()` gave a
  false positive rate of 0.296 and a true positive rate of 0.823.
