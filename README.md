# Evolutionary Sentence Convergence with Genetic Algorithms

[![Python](https://img.shields.io/badge/Python-3.8%2B-blue.svg)](https://www.python.org/)
[![JavaScript](https://img.shields.io/badge/JavaScript-ES6%2B-yellow.svg)](js/app.js)
[![HTML5/CSS3](https://img.shields.io/badge/Interface-Web%20%2B%20CLI-orange.svg)](index.html)
[![Dependencies](https://img.shields.io/badge/Dependencies-Zero%20(Pure%20Stdlib)-brightgreen.svg)](genetic_algorithm.py)
[![License: GPL-3.0](https://img.shields.io/badge/License-GPL%203.0-lightgrey.svg)](LICENSE)

An evolutionary computing simulation that evolves arbitrary target phrases from randomized character soup using **Genetic Algorithms**. Featuring both a zero-dependency **Python CLI engine** and an interactive **Browser Visualizer**, this project illustrates natural selection, fitness-proportionate selection, elitism, uniform crossover, and stochastic point mutation.

---

## 📸 Interactive Web Simulation

The repository includes a web-based visualizer allowing real-time inspection of evolutionary dynamics:

```
┌────────────────────────────────────────────────────────────────────────┐
│  Target Sentence: "Genetic Algorithms in Data Science"                 │
├────────────────────────────────────────────────────────────────────────┤
│  Population Size: [======|====] 1000                                   │
│  Elite Fraction:  [===|=======] 20%                                    │
│  Mutation Rate:   [=|=========] 5%                                     │
│  [ START SIMULATION ]   [ RESET ]                                      │
├────────────────────────────────────────────────────────────────────────┤
│  Generation: 48                                                        │
│  • Gen 00:  "X!kL-9s d@1pZ8A-9qP.14vB  "           (Fitness: 0.002)    │
│  • Gen 15:  "Gen#t!c Aldpr9tams i. D.tx"           (Fitness: 0.420)    │
│  • Gen 32:  "Genetic Algoritams in Data Sxience"   (Fitness: 0.884)    │
│  • Gen 48:  "Genetic Algorithms in Data Science"   (Fitness: 1.000) ✅ │
└────────────────────────────────────────────────────────────────────────┘
```

### Key Highlights
* **Zero External Dependencies:** The Python implementation runs on standard Python 3 (`string`, `random`), requiring no package installations. The browser interface runs in vanilla ES6 JavaScript without build tools or external frameworks.
* **Non-Linear Quadratic Fitness Metric:** Incorporates an exponential fitness scoring curve $F(\mathbf{g}) = \left(\frac{\text{matching}}{\text{length}}\right)^2$ that penalizes stagnant strings and magnifies selection pressure toward the global optimum.
* **Dual Crossover Strategies:** Implements both **Uniform Crossover** (independent 50/50 parental gene inheritance per locus) and **Single-Point Crossover** (structural block recombination).
* **Interactive Dynamic Controls:** Adjust population scale ($10$–$1000$), elite retention fraction ($0\%$–$50\%$), and stochastic mutation rate ($0\%$–$20\%$) in real time.

---

## 🧠 System Architecture & Genetic Pipeline

The evolutionary cycle operates through five continuous generational phases:

```
           [ User Input: Target String S_target ]
                             │
                             ▼
┌─────────────────────────────────────────────────────────┐
│ Phase 1: Population Initialization                      │
│ Generate N random DNA chromosomes of length L           │
└─────────────────────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────┐
│ Phase 2: Fitness Scoring                                │
│ Compute quadratic match proportion: F(g) = (matches/L)^2│
└─────────────────────────────────────────────────────────┘
                             │
            ┌────────────────┴────────────────┐
            ▼                                 ▼
   [ Fitness == 1.0? ]              [ Fitness < 1.0 ]
            │                                 │
     YES: TERMINATE                           │
  Target String Converged!                    ▼
                             ┌───────────────────────────────────┐
                             │ Phase 3: Elitism & Selection      │
                             │ Retain top E% fittest individuals │
                             │ Sample parents via Roulette Wheel │
                             └───────────────────────────────────┘
                                              │
                                              ▼
                             ┌───────────────────────────────────┐
                             │ Phase 4: Crossover Recombination  │
                             │ Uniform / Single-point crossover  │
                             └───────────────────────────────────┘
                                              │
                                              ▼
                             ┌───────────────────────────────────┐
                             │ Phase 5: Stochastic Mutation      │
                             │ Mutate characters with prob P_mut │
                             └───────────────────────────────────┘
                                              │
                                              ▼
                             [ Spawn Generation N + 1 ]
```

---

## 📐 Mathematical Formulation

### 1. Gene Pool & Chromosome Representation
Let the discrete alphabet $\Sigma$ consist of all alphanumeric characters, numbers, and common punctuation:
$$\Sigma = \{a\text{--}z, A\text{--}Z, 0\text{--}9, \text{space}, \text{", ', \&, :, ;, !, ., ,, -}\}$$

A candidate solution (individual DNA) $\mathbf{g}$ is represented as an ordered vector of $L$ character genes:
$$\mathbf{g} = [c_1, c_2, \dots, c_L], \quad c_i \in \Sigma, \quad L = |S_{\text{target}}|$$

### 2. Quadratic Fitness Function
Linear fitness scoring often suffers from weak selection gradients when comparing candidates with moderate similarity. To accelerate convergence and prevent premature drift, this pipeline squares the normalized matching ratio:
$$F(\mathbf{g}) = \left(\frac{1}{L} \sum_{i=1}^L \mathbb{I}(c_i = S_{\text{target}}[i])\right)^2$$
where $\mathbb{I}(\cdot)$ is the indicator function:
$$\mathbb{I}(\text{condition}) = \begin{cases} 1, & \text{if condition is true} \\ 0, & \text{otherwise} \end{cases}$$

*Example:* A candidate with 50% matching characters yields $F = 0.25$, whereas a candidate with 90% matching characters yields $F = 0.81$ (over **3.2x greater selection probability**).

### 3. Fitness-Proportionate (Roulette Wheel) Selection
Mating candidates are selected with probability directly proportional to their fitness:
$$P(\mathbf{g}_k) = \frac{F(\mathbf{g}_k)}{\sum_{j=1}^N F(\mathbf{g}_j)}$$

### 4. Uniform Crossover
During reproduction, child chromosomes inherit character alleles at each locus $i$ independently from either Parent A or Parent B:
$$c_i^{(\text{child})} = \begin{cases} c_i^{(A)}, & \text{with probability } 0.50 \\ c_i^{(B)}, & \text{with probability } 0.50 \end{cases}$$

### 5. Stochastic Point Mutation
To preserve diversity and prevent premature convergence into local minima, each gene has an independent mutation probability $P_{\text{mut}}$:
$$c_i' = \begin{cases} \text{random}(\Sigma), & \text{if } r \sim \mathcal{U}(0, 1) < P_{\text{mut}} \\ c_i^{(\text{child})}, & \text{otherwise} \end{cases}$$

---

## 📊 Evolutionary Dynamics & Tuning

| Parameter | Recommended Default | Impact of Setting Too Low | Impact of Setting Too High |
| :--- | :---: | :--- | :--- |
| **Population Size ($N$)** | `100` (CLI) / `1000` (Web) | Genetic bottleneck; premature stagnation | Higher memory and CPU cost per generation |
| **Elite Fraction ($E$)** | `20%` | High-fitness traits may be lost to mutation | Risk of cloning uniform individuals; reduced diversity |
| **Mutation Rate ($P_{\text{mut}}$)** | `5%` | Inability to introduce missing letters | Random walk behavior; genetic drift prevents convergence |

---

## 📁 Repository Structure

```
sentence-prediction-ga/
├── css/
│   └── style.css            # Stylesheet for browser visualizer
├── js/
│   └── app.js               # Browser GA implementation, sliders, and DOM rendering
├── genetic_algorithm.py     # Standalone Python CLI genetic algorithm engine
├── index.html               # Main interactive browser simulation interface
└── LICENSE                  # GNU General Public License v3.0
```

---

## ⚡ Quickstart

### Option 1: Running the Python Engine (CLI)

The Python implementation runs in any standard terminal without installing packages:

```bash
git clone https://github.com/FortunateSpy5/sentence-prediction-ga.git
cd sentence-prediction-ga

# Run script
python genetic_algorithm.py
```

* Enter any target sentence when prompted (e.g., `Hello, Genetic Algorithms!`).
* The script will output the best candidate string and fitness score for each generation until reaching 100% convergence ($F = 1.0$).

### Option 2: Running the Browser Visualizer (Web)

No server or npm installation is required:

1. Open `index.html` directly in any modern web browser (Chrome, Edge, Firefox, Safari).
2. Enter your desired sentence in the text area.
3. Configure the sliders for **Population Size**, **Elite Fraction**, and **Mutation Rate**.
4. Click **Start** to watch the characters self-organize across successive generations!

---

## 📄 License

This repository is distributed under the **GNU General Public License v3.0**. See the [LICENSE](LICENSE) file for complete details.