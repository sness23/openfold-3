# YouTube Script: OpenFold3 — Open-Source AlphaFold3

*Estimated runtime: 15–20 minutes*

---

## COLD OPEN (30 seconds)

[Screen: a protein structure rotating in 3D, colored by confidence]

"In 2018, one researcher watched Google solve a 50-year-old problem in biology — and asked, 'What just happened?' Seven years later, he built the open-source version. This is the story of OpenFold3."

---

## PART 1: THE PROBLEM (2–3 minutes)

[Screen: simple amino acid chain animation]

"Every living thing runs on proteins. Your muscles, your immune system, the enzymes digesting your food right now — all proteins. And here's the thing: a protein is just a chain of amino acids, like beads on a string. But that string folds into a specific 3D shape, and the shape IS the function."

[Screen: experimental lab footage or photos — X-ray crystallography, cryo-EM]

"For decades, if you wanted to know a protein's shape, you had to crystallize it, shoot X-rays at it, and spend months or years solving the structure. It cost tens of thousands of dollars per protein. Before 2020, we only knew the shapes of about 17% of human proteins."

"Scientists knew the sequence should determine the shape — Christian Anfinsen won the Nobel Prize for proving this in 1972 — but predicting that shape from the sequence? That was the 'protein folding problem,' and it was considered one of the grand challenges of biology."

---

## PART 2: ALPHAFOLD CHANGES EVERYTHING (3–4 minutes)

[Screen: CASP competition logos, DeepMind logo]

"Every two years, researchers compete in CASP — the Critical Assessment of Structure Prediction. Think of it as the Olympics for protein folding. In December 2018, a team from DeepMind entered with something called AlphaFold, and they won by a huge margin."

[Screen: blog post excerpt — "What just happened?"]

"Mohammed AlQuraishi, a computational biologist at Harvard, wrote a blog post that went viral in the science community. The title was 'AlphaFold @ CASP13: What just happened?' He described a 'broad sense of existential angst' among academic researchers — the fear that the best science would now only happen in corporate labs."

"Remember that name. AlQuraishi. He comes back."

[Screen: CASP14 results, GDT scores]

"Two years later, AlphaFold 2 entered CASP14 with a completely new architecture and essentially solved the problem. Median accuracy of 92.4 out of 100 — so accurate the competition organizers declared the protein folding problem 'largely solved.' AlQuraishi wrote another blog post: 'It feels like one's child has left home.'"

"DeepMind did something remarkable: they open-sourced AlphaFold 2 under Apache 2.0. Anyone could use it, modify it, build on it. By 2022, they'd predicted structures for 200 million proteins — nearly every known protein on Earth. Over 3 million researchers in 190 countries have used it. In October 2024, Demis Hassabis and John Jumper won the Nobel Prize in Chemistry."

---

## PART 3: ALPHAFOLD 3 AND THE CONTROVERSY (2–3 minutes)

[Screen: AlphaFold3 paper in Nature, molecular complex visualization]

"In May 2024, DeepMind published AlphaFold 3. This was a massive leap. AlphaFold 2 could only predict protein structures. AlphaFold 3 can predict proteins, DNA, RNA, small molecule drugs, ions — nearly everything in a biological cell, all at once."

"The architecture changed completely. Instead of predicting backbone angles, AlphaFold 3 uses a diffusion model — the same type of technology behind image generators like DALL-E. It starts with a cloud of random noise and gradually sculpts it into a 3D structure over 200 steps."

[Screen: AlphaFold3 license text]

"But here's where things got controversial. Remember how AlphaFold 2 was Apache 2.0 — fully open? AlphaFold 3's license is CC-BY-NC-SA — non-commercial only. You need Google's permission to use the model weights, even for academic research. The AlphaFold Server limits you to 20 predictions per day and won't let you predict how drugs bind to proteins."

"Meanwhile, DeepMind's spinoff Isomorphic Labs — which raised $600 million — uses AlphaFold 3 internally for commercial drug discovery. See the asymmetry? The most powerful tool in structural biology was now behind a corporate wall."

---

## PART 4: OPENFOLD3 — THE OPEN ALTERNATIVE (3–4 minutes)

[Screen: OpenFold GitHub page, AlQuraishi photo]

"Remember Mohammed AlQuraishi? The researcher who watched AlphaFold happen and asked 'What just happened?' He's now a professor at Columbia, and he's spent years building the open-source answer."

"OpenFold started as an open reproduction of AlphaFold 2. When AlphaFold 3 went closed, the OpenFold Consortium — backed by Bristol Myers Squibb, Novo Nordisk, Johnson & Johnson, NVIDIA, AWS, and others — built OpenFold3."

"Released in October 2025 as a preview, then updated in March 2026, OpenFold3 is a full, open-source reproduction of AlphaFold 3. Apache 2.0 license. You can use it for anything — academic research, commercial drug discovery, whatever you want."

[Screen: comparison table]

"But it's more than just the license. Here's what OpenFold3 gives you that AlphaFold 3 doesn't:

- Full training code. You can retrain the model from scratch or fine-tune it on your own data.
- Full training datasets — 300,000 experimental structures plus 13 million synthetic structures, all publicly available on AWS.
- A modular, hackable codebase in PyTorch.
- No prediction limits. Run as many jobs as your GPU can handle.
- Drug-protein interaction predictions — no restrictions."

"And it's competitive. OpenFold3 is the only open model that matches AlphaFold 3's performance on RNA structure prediction."

---

## PART 5: HOW IT WORKS (3–4 minutes)

[Screen: pipeline diagram — Input → MSA → Pairformer → Diffusion → Structure]

"Let me walk you through how OpenFold3 actually works."

"You give it an input — a protein sequence, maybe with a drug molecule and some DNA. The model processes this through several stages:"

[Screen: MSA visualization — aligned sequences]

"First, the MSA Module. MSA stands for Multiple Sequence Alignment. The model looks at thousands of related sequences from evolution. If two positions always mutate together, they're probably close in 3D space. This evolutionary signal is incredibly powerful — billions of years of natural experiments, compressed into a feature map."

[Screen: Pairformer blocks diagram]

"Next, the Pairformer. This is the model's main 'thinking engine' — 48 transformer blocks that reason about the relationships between every pair of residues. It builds a detailed spatial map of the protein."

"The whole trunk recycles 3 times — feeding its output back as input to iteratively refine its understanding."

[Screen: diffusion animation — noise to structure]

"Finally, the Diffusion Module. This is the magic. It starts with a cloud of random atoms — pure noise. Over 200 steps, it gradually denoises this cloud into a precise 3D structure. Think of it like a sculptor revealing a statue from marble."

"Because it's stochastic, you can run it multiple times and get different valid structures. By default, OpenFold3 generates 25 candidate structures and ranks them by confidence."

---

## PART 6: DEMO (3–4 minutes)

[Screen: terminal]

"Let me show you how easy it is to use. Three commands."

```
pip install openfold3
setup_openfold
run_openfold predict --query_json=examples/example_inference_inputs/query_ubiquitin.json
```

"That's it. `pip install`, run setup to download the model parameters, and predict."

[Screen: JSON input file]

"Here's what an input looks like. It's just JSON — you specify molecule type, chain IDs, and the sequence. For a drug, you give it a SMILES string — that's a text representation of a chemical structure."

[Screen: show protein-ligand JSON example]

```json
{
  "chains": [
    {"molecule_type": "protein", "sequence": "GDDELYR..."},
    {"molecule_type": "ligand", "smiles": "CC(=O)OC1C[NH+]2CCC1CC2"}
  ]
}
```

[Screen: prediction running in terminal]

"The model runs MSA generation, processes it through the Pairformer and diffusion module, and outputs structures."

[Screen: PyMOL/ChimeraX with predicted structure]

"Here's the output in PyMOL, colored by confidence. Blue means the model is very confident about this region — pLDDT above 0.9. Red means it's uncertain — often these are flexible loops or disordered regions. That uncertainty is useful information."

[Screen: overlay of predicted vs experimental structure]

"And when we overlay it with the experimental crystal structure — they're nearly identical. That's the power of this technology."

[Screen: protein-ligand complex visualization]

"Here's a protein-ligand prediction — you can see exactly where the drug molecule sits in the binding pocket. This is what drug discovery looks like in 2026."

---

## PART 7: THE LANDSCAPE (2 minutes)

[Screen: comparison table of models]

"OpenFold3 isn't alone. There's a whole ecosystem of open structure prediction models now:"

"**Boltz-2** from MIT and Recursion — uniquely predicts binding affinity, how tightly a drug binds, not just where it sits."

"**Chai-1** from Chai Discovery — offers a free web interface for commercial drug discovery."

"**Protenix** from ByteDance — strong for virtual drug screening."

"**ESMFold** from Meta — doesn't need MSAs, so it's much faster, though less accurate."

"What sets OpenFold3 apart is the full training pipeline. Most open models only release inference code — you can run predictions but you can't retrain the model. OpenFold3 released everything: training code, training data, the complete recipe."

---

## PART 8: WHAT'S NEXT (1–2 minutes)

[Screen: limitations list, future directions]

"These models aren't perfect yet. They predict static snapshots, but proteins are dynamic — they flex and breathe. Antibody-antigen prediction is still unreliable — critical for vaccines and immunotherapy. And predicting how tightly a drug binds, not just where it sits, remains a frontier."

"But the trajectory is clear. In 2018, this felt impossible. In 2020, it was solved for single proteins. In 2024, it expanded to all biomolecules. The field moves fast, and having open-source foundations like OpenFold3 means everyone can build on what came before."

---

## CLOSING (30 seconds)

[Screen: OpenFold GitHub, consortium logos]

"Seven years ago, Mohammed AlQuraishi asked 'What just happened?' Now he has an answer: we built the open-source version. And it belongs to everyone."

"OpenFold3 is on GitHub. Apache 2.0. If you have an A100, you can run it today. If you don't, there are free hosted options I'll link in the description."

"If you found this useful, like and subscribe. Links to everything in the description."

---

## DESCRIPTION BOX LINKS

```
OpenFold3 GitHub: https://github.com/aqlaboratory/openfold-3
OpenFold Consortium: https://openfold.io/
AlphaFold Server (free): https://alphafoldserver.com/
NVIDIA NIM (free prototyping): https://build.nvidia.com/openfold/openfold3
Chai Discovery (free): https://lab.chaidiscovery.com/
Neurosnap (free tier): https://neurosnap.ai/
AlQuraishi's "What Just Happened?" blog: https://moalquraishi.wordpress.com/2018/12/09/alphafold-casp13-what-just-happened/
OF3 Technical Report: https://portal.openfold.omsf.io/reports/of3p2_technical_report.pdf
```
