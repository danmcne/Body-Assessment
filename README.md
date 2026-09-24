# Body assessment

A spreadsheet (LibreOffice `.ods` and Excel `.xlsx`) and a Jupyter notebook that assess a person's body on five separately measured attributes:

| Attribute | What it measures | Target (score = 1) |
|---|---|---|
| **Leanness** | Waist-to-height ratio | 0.45 or lower (score 0 at 0.60) |
| **Fitness** | Estimated VO2max (median of the tests entered) | FRIEND 2022 90th percentile for sex, age and test mode; can exceed 1 |
| **Strength** | Relative strength across five lift categories, scored as Symmetric Strength does | Age-adjusted Wilks 400 in every category; can exceed 1 |
| **Size** | Lean mass against an upper reference (a population upper edge, not a limit) | Men: Casey Butt's frame-based estimate; women: FFMI 23.9; can exceed 1 |
| **Proportion** | Shape against ideal ratios (the waist is excluded; it counts only under leanness) | Classical ratios (men); hip and shoulder ratios (women) |

Each measurement counts once. Read the five scores as a **profile** first. They are also combined into a **weighted geometric mean**, so a weak attribute cannot be bought back by a strong one.

- **Weights:** you set them either by ranking **fitness-related health, looks and performance** (Purposes sheet) or by ranking the five attributes directly (Attributes sheet).
- **Ranges:** the composite carries a range from the body-fat and fitness-test errors.
- **Coverage:** the share of your chosen weight that has data. Below 100% the composite is marked **provisional**, since leaving out a weak attribute would otherwise raise it.

A **Plan** sheet then ranks what to work on:
- waist milestones with diet and cardio guidance;
- fitness targets by percentile;
- which lift categories and body parts to prioritise;
- a hard / medium / maintenance tier for each muscle group;
- an exercise list ordered by priority, marked compound or isolation.

The workbook supports **men and women**. The female assessment has additional limitations, listed [below](#limitations).

> This is a self-assessment tool, not medical advice. Consult a physician before starting a new diet or training programme, particularly with a medical condition.

## Files

| File | Purpose |
|---|---|
| `body_assessment.ods` | Workbook, LibreOffice format |
| `body_assessment.xlsx` | The same workbook, Excel format |
| `body_assessment.ipynb` | Optional, for Python users: a second implementation of the same rules, with plots, measurement history, a lever ranking and a cross-check against the workbook. The workbook is the tool and the source of truth |

## Quick start

1. Open the workbook. Yellow cells with blue text are inputs; everything else is a formula.
2. On **Measurements**, fill one column per date: sex, age, height, weight, girths, lifts and any fitness test.
   - The example column holds a typical recreational woman. Overwrite or delete it.
   - Leave unmeasured cells blank; missing inputs blank out only the rows that need them.
3. On **Summary**, cell B3 chooses whether the **Purposes** or the **Attributes** ranking drives the composite. Set your ranks (1 = most important, ties allowed) or type points.
4. Read **Summary**, **Report**, **Strength** and **Plan**.

The analysis uses the latest dated column. To analyse an earlier column, type its date in `Report!B2`.

### Notebook

```bash
pip install pandas numpy matplotlib openpyxl odfpy jupyter
jupyter notebook body_assessment.ipynb
```

- Set `WORKBOOK` in the first code cell to the `.xlsx` or `.ods` file.
- The notebook holds no data or constants of its own. It reads everything from the workbook, recomputes every result with its own code, and reports whether the two agree.
- This catches implementation errors in either version, not errors in the shared rules.
- Save the workbook in Excel or LibreOffice before running, so the cached values exist for the cross-check.

## Measuring

- **Waist**
  - Men: horizontal at the navel.
  - Women: at the narrowest point.
  - Both: standing, at the end of a normal exhalation, belly neither pulled in nor pushed out.
- **Hips:** widest point of the buttocks. Required for women.
- **Wrist and ankle:** smallest points, at the hand side of the wrist bone and just above the ankle bones.
- **Girths:**
  - Casey Butt's model uses relaxed chest (nipple level), relaxed mid-thigh and relaxed calf, with a flexed arm and a clenched-fist forearm.
  - The classical male ratios use flexed or largest girths.
  - Both sets of rows are provided.
- **Lifts**
  - Load is the total weight moved: bar, plates and collars.
  - For chin-ups, pull-ups and dips, enter the added weight (negative if assisted).
  - Reps are to failure, whole numbers 1–10; other rep counts are not scored.
  - **Leg press: 45° sled machines only**, load = sled plus plates. Horizontal, vertical and lever leg presses are not comparable.
  - Gym bars and plates are often not their nominal weight. Weigh the bar by stepping on a bathroom scale with and without it, and weigh small plates on a kitchen scale.
- **Fitness tests:** Rockport 1-mile walk (time and finish heart rate), Cooper 12-minute run, or a cycle ramp test with peak watts. With several tests, the median score is used.
  - Race times (5 km etc.) are **not** accepted: Daniels' VDOT measures running performance, not VO2max.
  - Swimming and elliptical results are not comparable and are not accepted.

## Sheets

| Sheet | Contents |
|---|---|
| Read me | Instructions and limitations |
| Summary | Attribute scores with ranges, composite, coverage, weighting switch |
| Attributes | Rank the five attributes directly; read-only purpose view of that ranking |
| Purposes | Rank fitness-related health, looks and performance; the effective attribute weights that ranking implies |
| Plan | Waist milestones, cardio targets, strength and size priorities, muscle-group tiers, exercise ranking |
| Report | Body composition with error ranges, fitness, proportion, left/right symmetry, size upper references, maximum girths |
| Strength | Estimated 1RMs and rep maxes, scores, tiers, strength balance, estimated powerlifting total and Wilks, muscle groups, charts |
| Measurements | All inputs, one column per date |
| Parameters | Every constant with its source (or marked as a judgment): shared, sex-specific (male / female / in use), ratios, lift factors, equipment and confidence per lift, norms, exercise library |

## Method in brief

- **Body fat:** the US Navy circumference equations; the female version adds hips. The formula's standard error (3.5 percentage points for men, 3.7 for women) is carried into lean mass, the size score and the composite as a range.
- **Fitness:** each test's VO2max estimate is divided by the FRIEND 90th percentile for your sex, age decade and test mode, and the median across tests is used.
  - Percentiles are interpolated only within FRIEND's published 10th–90th range; nothing is extrapolated.
- **Strength**
  - One-rep maximum from Wathan's formula.
  - Each lift is converted to a deadlift equivalent via lift ratios.
  - A lift's score is a quarter of the age-adjusted Wilks score of the powerlifting total it implies. This is Symmetric Strength's published definition.
  - The composite uses the geometric mean of the five category scores, so imbalance lowers the result. **Strength balance** is 100 × geometric mean ÷ arithmetic mean. "Symmetry" is kept for left/right differences.
  - Every lift has a confidence class: high for the benchmark lifts, medium for converted variants, low for machines. A machine lift (45° sled leg press, lat pulldown, seated cable row) counts for its category only when no free-weight lift is entered there; it always counts for the muscle groups.
  - Belt squats are not included: machine designs vary too much for a usable conversion.
  - Muscle-group scores are **inspired by** Symmetric Strength. They use its lift lists with equal weights, because its weights are unpublished.
- **Size:** lean mass (weight minus Navy fat) divided by an upper reference.
  - Men: Casey Butt's formula from height, wrist and ankle.
  - Women: FFMI 23.9 × height².
- **Proportion:** the root-mean-square of the log deviations from the ideal ratios, reported as exp(−RMS).
- **Plan**
  - Lever sensitivities in the notebook rerun the whole model with one input changed.
  - Tiers are rule-based. Thresholds and set ranges are editable on Parameters.

## Limitations

**General**

- Tape-based body fat has a standard error of about 3–4 percentage points. Field-test VO2max estimates carry roughly ±5 ml/kg/min.
- The size references (Butt, Berkhan, FFMI 25 for men, FFMI 23.9 for women) are upper edges of particular athlete samples, not biological limits. Scores above 1 are possible.
- "Health" means **fitness-related** health only. There are no blood pressure, blood markers, activity or disease data.
- Only the body-fat and fitness-test errors are propagated. The 1RM-estimate error (about 5%) is not.
- Classical proportion ideals are conventions, not health standards.
- Some lift ratios are marked `ESTIMATE` or `USER ASSUMPTION` on Parameters: sumo deadlift, snatch press, pull-up, lat pulldown, seated cable row and leg press. Scores for those lifts will differ from symmetricstrength.com.
- Several values are heuristics, labelled as such and editable:
  - the 10% strength-classification tolerance;
  - weekly set ranges;
  - interval and protein guidance;
  - rank-to-weight mapping (1/rank); type points for exact control;
  - target body fat;
  - the 0.45 waist-to-height endpoint. This is a physique target; the health bands treat the whole 0.4–0.5 range as fine.

**Additional limitations of the female assessment**

- **Size:** Casey Butt's formula and maximum girths exist for men only.
  - Women are scored against a height-only FFMI upper reference of 23.9, the 97.5th percentile of drug-tested female college athletes. In that study, rugby players reached 25.8 and the highest value observed was 27.2.
  - Symmetric Strength uses 19.2 instead.
  - Frame size is ignored.
- **Proportion:** there are no classical female limb ideals. Only two ratios are scored:
  - hips relative to height, implied by a waist/hip ratio of 0.70 at the leanness target. The WHR finding is contested and based mostly on men's ratings of attractiveness.
  - shoulders ≈ hips. This is a judgment made by analogy with the "hourglass" definition.
  - Treat the proportion score as a weak signal, or give it 0 points.
- **Plan:** size tiers exist only for glutes, adductors and deltoids. Other groups are tiered from strength alone.
- **Strength:**
  - The squat and bench ratios come from Symmetric Strength.
  - The overhead press and row ratios come from Strength Level.
  - The push-press and chin-up factors are scaled estimates.
  - The leg-press rule is untested for women.
- **Body fat and leanness:** the female Navy formula needs a natural-waist measurement and hips. The 22% target body fat is a convention.
- **Fitness:** FRIEND norms are US data. The Cooper formula was derived in men.

Default rankings are neutral for both sexes. Nothing in the workbook assumes what anyone should value.

## Sources

### Body composition and leanness
- Hodgdon JA, Beckett MB. *Prediction of percent body fat for U.S. Navy men and women from body circumferences and height.* Naval Health Research Center, San Diego, 1984 (Reports 84-11 and 84-29).
- Institute of Medicine. *Assessing Readiness in Military Women: The Relationship of Body Composition, Nutrition, and Health.* National Academies Press, 1998 (Navy-equation standard errors).
- Ashwell M, Gibson S. Waist-to-height ratio as an indicator of "early health risk". *BMJ Open* 2016;6:e010159.
- American Council on Exercise. Body-fat percentage categories (source of the 22% female target-body-fat convention).

### Size upper references and girths
- Butt C. *Your Maximum Muscular Bodyweight and Measurements* (weightrainer.net) and *Your Muscular Potential*.
- Berkhan M. Maximum muscular potential of drug-free athletes. leangains.com, 2010.
- Kouri EM, Pope HG, Katz DL, Oliva P. Fat-free mass index in users and nonusers of anabolic-androgenic steroids. *Clin J Sport Med* 1995;5(4):223–228.
- Harty PS et al. Upper and lower thresholds of fat-free mass index in a large cohort of female collegiate athletes. *J Sports Sci* 2019;37(20).
- Schutz Y, Kyle UUG, Pichard C. Fat-free mass index and fat mass index percentiles in Caucasians aged 18–98 y. *Int J Obes* 2002;26:953–960 (via Symmetric Strength's 19.2 estimate).

### Proportion
- Reeves S. *Building the Classic Physique: The Natural Way.* 1995.
- McCallum J. "The Keys to Progress" series, *Strength & Health* (1960s).
- Singh D. Adaptive significance of female physical attractiveness: role of waist-to-hip ratio. *J Pers Soc Psychol* 1993;65(2):293–307.
- Tovée MJ, Maisey DS, Emery JL, Cornelissen PL. Visual cues to female physical attractiveness. *Proc R Soc B* 1999;266:211–218 (the counter-evidence on WHR).
- Simmons KP, Istook CL, Devarajan P. Female Figure Identification Technique (FFIT) for apparel. *J Textile Apparel Technol Manag* 2004;4(1).

### Strength
- Symmetric Strength, "About": strength score definition, category averaging, muscle-group lift lists, and squat/bench ratios for men (0.87 / 0.65) and women (0.84 / 0.57). symmetricstrength.com/about
- SensAI, "Strength Standards by Bodyweight, Sex & Age" (Symmetric Strength tier thresholds and intermediate standards, and Strength Level ratios). sensai.fit
- Strength Level, strength standards (female overhead press and row ratios). strengthlevel.com
- Thibaudeau C / Poliquin C structural-balance ratios (front squat, power clean, incline, dip, chin-up). thibarmy.com
- LeSuer DA, McCormick JH, Mayhew JL, Wasserstein RL, Arnold MD. The accuracy of prediction equations for estimating 1-RM performance in the bench press, squat, and deadlift. *J Strength Cond Res* 1997;11:211–213 (Wathan formula).
- Wilks R. Wilks formula, original coefficients for men and women; Vanderburgh PM, Batterham AM. Validation of the Wilks powerlifting formula. *Med Sci Sports Exerc* 1999;31(12):1869–1875.
- Foster and McCulloch age coefficients, USA Powerlifting age-coefficient table.

### Fitness
- Kaminsky LA et al. Updated reference standards for cardiorespiratory fitness measured with cardiopulmonary exercise testing: data from FRIEND. *Mayo Clin Proc* 2022;97(2):285–293. Open access, CC BY-NC-ND 4.0. Table 3 percentiles are reproduced as numerical data with attribution. The 90th percentile is the fitness reference.
- Kline GM et al. Estimation of VO2max from a one-mile track walk, gender, age, and body weight. *Med Sci Sports Exerc* 1987;19(3):253–259.
- Cooper KH. A means of assessing maximal oxygen intake. *JAMA* 1968;203:201–204.
- American College of Sports Medicine. *ACSM's Guidelines for Exercise Testing and Prescription* (leg-cycling metabolic equation).
- Kodama S et al. Cardiorespiratory fitness as a quantitative predictor of all-cause mortality and cardiovascular events. *JAMA* 2009;301(19):2024–2035.

### Plan guidance
- World Health Organization. *Guidelines on physical activity and sedentary behaviour.* 2020.
- Helgerud J et al. Aerobic high-intensity intervals improve VO2max more than moderate training. *Med Sci Sports Exerc* 2007;39(4):665–671.
- Morton RW et al. A systematic review, meta-analysis and meta-regression of the effect of protein supplementation on resistance training-induced gains. *Br J Sports Med* 2018;52:376–384.
- Schoenfeld BJ, Ogborn D, Krieger JW. Dose-response relationship between weekly resistance training volume and increases in muscle mass. *J Sports Sci* 2017;35(11):1073–1082.
- Bickel CS, Cross JM, Bamman MM. Exercise dosing to retain resistance training adaptations in young and older adults. *Med Sci Sports Exerc* 2011;43(7):1177–1187.
- Leong DP et al. Prognostic value of grip strength. *Lancet* 2015;386:266–273.

## License

The workbook, notebook and documentation are released under the [MIT License](LICENSE).

**Third-party material.** The repository reproduces no text, figures or code from the sources above. It uses published formulas, coefficients and small tables of numerical results, each attributed on the Parameters sheet and listed here. Formulas and numerical facts are generally not protected by copyright, so their use is compatible with the MIT License.

One source is itself under a restrictive licence: the FRIEND percentiles (Kaminsky et al. 2022, CC BY-NC-ND 4.0). Only the numbers are reproduced, with attribution. If you need certainty for commercial use, check this yourself or replace that table on the Parameters sheet.

This project is not affiliated with or endorsed by Symmetric Strength, Strength Level, SensAI, Casey Butt or any other source listed.
